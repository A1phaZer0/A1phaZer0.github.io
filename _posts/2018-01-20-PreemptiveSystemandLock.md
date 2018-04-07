---
title: "Preemptive System and Lock"
author: A1phaZer0
layout: post
category: Kernel
---

> **Preemptive System**

Most modern operating systems are **preemptive system**. Preemption is a way that interrputs running process/task without its coorperation.

> **Spin Lock**

**Spin Lock** is a way for proventing critical resources/objects from being modified by multiple entities at the same time. The basic logic behind spin lock is:
```c
cli();
while(spin-lock-not-available) {} // Wait for the lock
// Critical Section
sti();
```

<!--more-->

On **uniprocessor(UP)** system: 

**Spin Lock** is a no-op. Because after **cli()**, timer-IRQ is also blocked, no scheduling will happen unless critical section access finished.

If no interrupt handler uses same spin lock, **cli()/sti()** are not necessary, process holding the lock can be preempted, but since it still holds the lock, other process will wait and be re-scheduled.

On **SMP** system:

**cli()/sti()** are for enabling/disabling interrupts on current cpu(core).

