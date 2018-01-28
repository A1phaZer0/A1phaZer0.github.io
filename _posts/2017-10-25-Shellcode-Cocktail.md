---
title: "Shellcode Cocktail"
layout: post
author: A1phaZer0
category: HackING
---

> ### Function call

> #### Function call

> **Function call**

**> Function call**

** > Function call**

In at&t syntax:
```cpp
// jump to address stored in eax
call *eax
// jump to address stored in memory location that eax point to
call *(eax)
// like above
call *0xdeadbeef
```
> #####Relative jump

Relative jump will take control flow to somewhere from current `eip`. After instruction fetch stage, current `eip` will point to instruction next to `jmp`, so the real distance is calculated by `addr of des - addr of inst next to jmp`.  
```c
EB cb: jmp rel8
```
`cb/rel8` is a signed value.

> #####Breakdown of shellcode

**jmp-call-pop method**    
GAS:  
Add ` .intel_syntax noprefix` to use intel syntax and without `%` before registers.
```cpp
.intel_syntax noprefix
.section .text
.global _start

_start:
	jmp LABELX
LABELY:
	pop esi
    
	/* craft what you want */

LABELX:
	call LABELY
	.ascii "/bin/shAAAABBBBCCCC"
```
NASM:
```cpp
section .text
global _start

_start:
	jmp LABELX
LABELY:
	pop esi
    
	/* craft what you want */

LABELX:
	call LABELY
	db '/bin/shAAAABBBBCCCC'
```
**stack method**  
```cpp
+=================+
|      NULL       |
+=================+
|    '//bin/sh'   |
+=================+ <- ebx
|      NULL       |
+=================+ <- edx
|      ebx        |
+=================+ <- ecx
```
Why `'//bin/sh'`? Because it's 8 characters, one can use `push 0x68732f6e` and `push 0x69622f2f` to craft and no zero introduced.

> #####Disassemble specific function by gdb

```bash
$ gdb -batch target_bin -ex 'disas main'
# -ex to run command in gdb
```

> #####Generate shellcode

Compile with gcc
```bash
$ gcc -nostdlib -m32 -o target source.S
```
Compile with nasm
```bash
$ nams -f elf source.asm
$ ld -m elf_i386 -o target source.o
```
Generate shellcode
```bash
# get disassembly code in intel syntax
# remove labels
# get second column (opcode)
$ objdump -d target_bin -M intel | grep '^ ' | cut -f2
# add \x to each opcode and remove \n
$ for i in $( objdump -d bin - M intel | grep '^ ' | cut -f2 ); do echo -n '\x'$i; done; echo;
```

> #####Test shellcode

```cpp
#include <stdio.h>
/* shellcode goes here */
char shellcode[] = "\xef\xbe\xad\xde"
		   "\xef\xbe\xad\xde";
int main()
{
    int *ret;
    /* cast shellcode to function pointer then invoke */
    ((void (*)())shellcode)();
    /* or use address of shellcode to override ret address of main */
    /* shellcode will be called when main() return . */
    ret = (int *)&ret + 2;
    *ret = (int)shellcode;
}
```
```bash
$ gcc -m32 shellcode.c -o shellcode -fno-stack-protector -z execstack
```
