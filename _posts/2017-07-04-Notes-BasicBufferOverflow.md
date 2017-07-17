---
title: "Notes-BasicBufferOverflowTechniques-0x2"
author: A1phaZer0
layout: post
category: HackING
---
### Heap Overflow
Like stack, heap also could be overwrote.This notes is about heap corruption related things.  
<!--more-->
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
|         |         "\x00\x00\x00\x00AAAA\x01\x00\x00\x00%x%9x%x%n%17x%n"
|         |          |_________________________________________| |   | |
|         |                               |                      |   | |
|         |                               V                      |   | |
|         |          number of characters so far (4+4+4+1+9+1=23)|   | |
|         |                                                      V   | |
|         |                                          write 0x17 to   | |
|         |                                             0x00000000   | |
|         |                                                          V |
|         |                                               read 4 bytes |
|         |                                                       AAAA |
|         |                                  print it as 17 chars long |
|         |                                                            V
|         |                                                write 0x28 to
|         |                                                   0x00000001
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