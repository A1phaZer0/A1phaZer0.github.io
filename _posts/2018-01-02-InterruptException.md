---
title: "Interrupt and Exception"
author: A1phaZer0
layout: post
category: Kernel
---

> **Interrupt**

Interrupts can be produced by hardware or software. `IF` in the EFLAGS is used to block hardware interrupts, software interrupts can't be blocked. 

Unlike exception, there's no error code pushed onto stack when control transfered to interrupt handler.

> **Exception**

Exceptions are coming from software, either when some error happens or be produced on purpose.

There are 3 types of exceptions.
1. Fault: normally can be corrected, the instruction when fault happened will run again.
2. Trap: when trap handler finished it's job, control will be transfered back to next instruction after the instruction generating trap.
3. Abort: something fatal happened, there's no going back.

<!--more-->

> **Interrupt Descriptor Table**

Three types of gates can be in IDT: Interrupt Gate, Trap Gate, Task Gate.

```bash
Interrupt Gate
31                            16               v       4       0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|OFFSET IN SEGMETN 31~16        |P|DPL|0 1 1 1 0|0 0 0|         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
31                            16                               0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|SEGMENT SELECTOR               |OFFSET IN SEGMENT 15~0         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Trap Gate
31                            16               v       4       0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|OFFSET IN SEGMETN 31~16        |P|DPL|0 1 1 1 1|0 0 0|         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
31                            16                               0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|SEGMENT SELECTOR               |OFFSET IN SEGMENT 15~0         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Task Gate
31                            16                 7             0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                               |P|DPL|0 0 1 0 1|               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
31                            16                               0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|SEGMENT SELECTOR               |                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

According to control transfering, stack switching could happen.

```bash
+-------------+ +
|      |  SS  | |
+------+------+ Pushed if stack switched
|     ESP     | |
+-------------+ +
|   EFLAGS    |
+------+------+
|      |  CS  |
+------+------+
|     EIP     |
+-------------+
| ERROR CODE  | Pushed if it's a exception
+-------------+
```

To exception, error code will be push onto stack, but to interrupt, there's no error code.

> **Error Code**

```bash
Error Code
31                            16                               0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                               |                         |T|I|E|
|                               |SELECTOR INDEX           | |D|X|
|                               |                         |I|T|T|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                                                             | |
                                                             | +-> External
                                                             |     Event
                                                             +---> IDT or
                                                                   GDT/LDT

Page-fault Exception Error Code
31                            16                               0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                         |U|R| |
|                                                         |/|/|P|
|                                                         |S|W| |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

