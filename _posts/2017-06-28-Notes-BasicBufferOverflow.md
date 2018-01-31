---
title: "Notes-BasicBufferOverflowTechniques-0x1"
layout: post
author: A1phaZer0
category: Hacks
---

OSes on various architectures have different layouts of memory.
<!--more-->

```
Typical memory layout of Linux on 32-bit x86 architecture.
+=============+ 0xffffffff
|             |
|             |
|             | kernel space
|             |
|             |
|             |
+=============+ 0xc0000000       +=============+
|             | <-- stack top    |    NULL     |   4
|    env      |                  |    NULL     | Bytes
|             |                  |    NULL     | NULLs
|             |                  |____NULL_____|
|             |                  |  NULL('\0') | <- string terminator
|             |     0xbffffffa-> |      X      | <- last char of program's
|             |                  |             | name, absolute path or 
/             / user space       |_____________| relative path.
\             \                  |      \0     |
|             |                  |      X      |  environment variable:
|             |                  |             |  _=program name
|             |                  |_____________|
|             |                  |             |
|             |                  | other envs  |
|             |                  | & shellcode |
|             |                  |_____________|
|             |                  |   "argN"    |<--+
|             |                  |     .       |   |
|             |                  |     .       |   |
|             |                  |   "arg1"    |   |
|             |                  |program name |   |
|             |                  |             |   |
|             |                  |             |   |
|             |                  |   argv[N]   |---+
|             |                  |     .       |
|             |                  |     .       |
|             |                  |   argv[1]   |
|             |                  |   argv[0]   |
|             |                  |____argc_____|
|             |                  |             |
|             |                  |             |
|             |                  |             |
|             |                  +=============+
|             |
|             |
+=============+ 0x00000000
```
>Name of program has influence on address of shellcode, one character increase in
name length makes two bytes decrease in shellcode address.  
>It can't be checked by gdb, since gdb sets <span style="color:red">**_**</span> environment variable to path
of gdb, and puts this variable under shellcode.

Use **execl()** or **execle()** to exploit target program.  

**execl()** replaces current process with target program.

```c

int execl(const char *path, const char *name, const char *args, NULL);

```

**execle()** just like **execl()**, but it could be given a new environment.

```c

int execle(const char *path, const char *name, const char *args, NULL,  

		           char *const envp[]);

           

char shellcode[] = "...";

char *env = {shellcode, NULL}; //Must be NULL ended.

```
