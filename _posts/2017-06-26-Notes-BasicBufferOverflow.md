---
title: "Notes-BasicBufferOverflowTechniques"
layout: post
author: A1phaZer0
category: HackING
---

```c
void f2(int a, int b)
{
	int s;
	int k[5];
	a = 0;
}
void f1(int c, int d, int e)
{
	f2(c, d);
}
```
```
+==============+ HIGH ADDRESS
|              |
|      e       |        |
|      d       | STACK  |
|      c       |        V
|   ret addr   |
|   older ebp  |---+ <- ebp of f1
|      d       |   |                                     | ret addr |
|      c       |   | Func1          rewrite ret addr-\   | ret addr |
|   ret addr   |---+              |overflow|<--------/   | ret addr |---------+
| ebp of Func1 |---+ <- ebp of f2 |overflow|             | ret addr |         |
|      s       |   |              |overflow|<- rewrite s | ret addr |         |
|     k[4]     |   |              |overflow|             |   \xXX   |---+     |
|     k[3]     |   | Func2        |overflow|             |   \xXX   |   |     |
|     k[2]     |   |              |overflow|             |   \xXX   |shellcode|
|     k[1]     |   |              |overflow|             |   \xXX   |   |     |
|     k[0]     |---+              |overflow|             |   \xXX   |---+     |
|              |                                     +---|    NOP   |         |
+==============+                                     |   |    NOP   |         |
|              |                                 NOP sled|    NOP   |         |
|     ...      |                                     |   |    NOP   |<--------+
|              |                                     +---|    NOP   |
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
