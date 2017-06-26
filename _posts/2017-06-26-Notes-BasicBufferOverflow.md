---
title: Notes-BasicBufferOverflowTechniques
layout: post
author: A1phaZer0
category: HackING
---
#Basic buffer smash techniques on Linux

#####C Calling convention on x86

```c
void Func2(int a, int b)
{
	int s;
    int k[5];
    a = 0;
}
void Func1(int c, int d, int e)
{
	Func2(c, d);
}
```
```
+==============+ HIGH ADDRESS
|              |
|      e       |        |
|      d       | STACK  |
|      c       |        V
|   ret addr   |
|   older ebp  |---+ <- ebp of Func1
|      d       |   |                                          | ret addr |
|      c       |   | Func1              rewrite ret addr-\    | ret addr |
|   ret addr   |---+                  |overflow|<--------/    | ret addr |---------+
| ebp of Func1 |---+ <- ebp of Func2  |overflow|              | ret addr |         |
|      s       |   |                  |overflow|<- rewrite s  | ret addr |         |
|     k[4]     |   |                  |overflow|              |   \xXX   |---+     |
|     k[3]     |   | Func2            |overflow|              |   \xXX   |   |     |
|     k[2]     |   |                  |overflow|              |   \xXX   |shellcode|
|     k[1]     |   |                  |overflow|              |   \xXX   |   |     |
|     k[0]     |---+                  |overflow|              |   \xXX   |---+     |
|              |                                          +---|    NOP   |         |
+==============+                                          |   |    NOP   |         |
|              |                                      NOP sled|    NOP   |         |
|     ...      |                                          |   |    NOP   |<--------+
|              |                                          +---|    NOP   |
+==============+
|              |
|              |        A
|              | HEAP   |
|              |        |
+==============+
|              |
+==============+
|    .bss      |
+==============+
|    .data     |
|              |
+==============+
|              |
|    .text     |
|              |
+==============+
|              |
|              |
|              |
+==============+ LOW ADDRESS
```
