---
title: "Hardware Architecture"
author: A1phaZer0
layout: post
category: Kernel
---

> **CPU, MCH, ICH**

MCH(**M**emory **C**ontroller **H**ub) also known as Northbridge.
ICH(**I**/O **C**ontroller **H**ub) also known as Southbridge.

High speed componets connect together via MCH, while low speed componets via ICH.

There are three types of bus.
1. Address bus, for addressing.
2. Data bus, for data transmission.
3. Control bus, for control signals.

CPU connects with each componet via buses and MCH/ICH.

<!--more-->

> **I/O addressing**

I/O componets on IBM PC are using a separate address space, from 0x0000 to 0xffff. Kernel can access these componets by addresses(ports).

> **BIOS**

```bash
                +===============+                         -------
                |               |<- 0xFFFFFFFF               ^
       jmp +----|               |                            |
           |    |               |                             
           |    |    ROM BIOS   | reserved for BIOS, 64KB    |
           |    |               |                             
           +--->|               |                            |      
                |               |<- 0xFFFF0000                
                +---------------+                      BIOS program
        +-------|               |                             
        |       |               |                            |
        | copied|               |                             
        |   to  |               |                            |
        |       |               |                             
        |       |               |                            |
        |  +----|               |                            v
        |  |    +---------------+                         -------
        |  |    \               \
        |  |    /               /
        |  |    +---------------+
        |  +--->|///////////////|<- 0xFFFFF
        |       |///////////////|
        |       |///////////////|
        |       |///////////////| ROM BIOS shadow
        |       |///////////////|
        |       |///////////////|
        +------>|///////////////|<- 0xF0000
                +---------------+
                \               \
                /               /
                +---------------+
                |               |
                |               |<- tables created by BIOS
                |               | (interupt table, system state)
                +===============+
```
**NOTEs:**
1. `jmp` for switching different CPU addressing mode.
2. Part of BIOS is copied to BIOS shadow area, some tables, system information, system initialization are done.
3. Jump to BIOS shadow, from now on, CPU works at `REAL MODE`.
4. Finally, BIOS will load boot loader(boot sector) at 0x7c00, jump here and go.

> **A20**

The A20 (counting from 0) address line represents the 21st bit of memory address. It's disabled on boot, which means address beyond 1MB gonna be truncated.


A20 can be enabled via `keyboard controller` or `fast A20 gate`.
