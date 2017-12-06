---
title: "Notes-BasicBufferOverflowTechniques-0x2"
author: A1phaZer0
layout: post
category: HackING
---
HEAP OVERFLOW, FORMAT STRING, PLT & GOT, etc.
<!--more-->
### Heap Overflow
Like stack, heap also could be overwrote.This notes is about heap corruption related things.  
Overview of heap:
```
+=========+ <- HIGH ADDRESS
|         |
|  chunk  |
|         |
|_________|
|__header_|
|         |
|  chunk  |
|         |
|_________|
|__header_|
```
### Format String Vulnerability
#### Theory
The format string vulnerablility comes from the way that format string being interpreted.  
- When printf meets %s, the value of corresponding argument will be interpreted as memory address of a string, then this string will be printed.  
- When printf meets %n, the number of characters printed so far will be written to the memory block that the corresponding argument points to.

But, what if no argument exists?   **OVERFLOW**

```c
+=========+ <- HIGH ADDRESS
|         |
|         | char fmt_str1[50] = "\xef\xbe\xad\xde%x%x%x%s";
|         |                        0xdeadbeef    ^ ^ ^ ^ 
|         |                                      | | | |
|         |                               ebp + 12 | | |
|         |                                 ebp + 16 | |
|         |                                   ebp + 20 |
|         |                                     ebp + 24
|         |
|         | char fmt_str2[50] = 
|         |         "\x04\x80\x04\x08AAAA\x08\x80\x04\x08%8x%9x%10x%n%17x%n"
|         |          |____________________________________________| | | |
|         |                                |                        | | |
|         |                                V                        | | |
|         |          number of characters so far (4+4+4+8+9+10=39)  | | |
|         |                                                         V | |
|         |                                             write 0x27 to | |
|         |                                                0x08048004 | |
|         |                                                           V |
|         |                                                read 4 bytes |
|         |                                                      "AAAA" |
|         |                                   print it as 17 chars long |
|         |                                                             V
|         |                                                 write 0x38 to
|         |                                                    0x08048008
|         |
|_fmt_str_| <- address of format string, ebp + 8, as first argument of printf.
|___ret___|
|_old_ebp_| <- ebp
|         |
|         |
+=========+
```
> If fmt_str1 is located near printf's calling stack, like `ebp + 24`, then printf will step through memorys by `%x`s then use `0xdeadbeef` as string address and print it.  

> The `%17x` in fmt_str2 means to control number of characters printed so far, so put 4 bytes junk data "AAAA" before next address to write is necessary.  

> <span style = "color: red"> **NO \x00 IN THE STRING, IT'S A NULL.** </span>

#### Direct Parameter Access
```c
printf("%4$x");
```
This function will print 4th argument of printf, the format string(pointer) itself is `0th`, but 0 can't be in a parameter.
```c
printf("%2$x %1$llx");
// or
printf("%1$llx %2$x");
```
On IA-32 platform, 2nd argument is located 4 bytes after 1st one, but if we use `llx` in parameter, the location of 2nd argument will be 8 bytes after 1st one.  
> Direct parameter access can be mixed with usual parameters if they step same size of memory, like `printf("%2$x %x");`, both of them step 4 bytes memory. Otherwise, like `printf("%2$llx, %x");`, will cause some kind of inconsistency (Ubuntu 7.04, gcc 3.3.6).

#### Short Writes
Because the bigest short int is `0xFFFF`(65535), means that it could be used to construct any number by `printf` with `%hn`, just split a number into two half.
```c
//0x0010bfff
//bfff(16) == 49151(10)
printf("\x06\x01\x01\x01JUNK\x04\x01\x01\x01%49143x%5$hn%16401x%6hn");
//      |           8 + 49143 = 0xbfff          |     |0x10010-0xbfff|
// write 0x0010bfff to location 0x01010104
```

#### .ctors and .dtors
Constructors and destructors are two special writable sections in ELF, `.ctors` is for functions to execute before entering program's real execution stage, `.dtors` is quite the opposite.
```c
+==========+ <- HIGH ADDRESS
|0x00000000| <-- __DTOR_END__
|          | <-- destructor function address, doesn't exist if there is none.
|0xffffffff| <-- __DTOR_LIST__
+==========+
If *__DTOR_END__ is rewritten to a function address, 
that function will be executed after exit(0);
```

#### PLT & GOT
```asm
<func@plt>:
jmp *0xaabbccdd  --> 0xaabbccdd is point to a slot of GOT 
push 0x0
jmp 0xccddeeff   --> for lazy-binding
```
Changed target GOT slot will take control flow to any function.  

**Notice: Before a instruction's execution, eip is pointing this instruction, when this instruction is being processed, eip is pointing to next instruction(not the one jmp or call set), after this instruction's execution, eip is set to the target address.**
