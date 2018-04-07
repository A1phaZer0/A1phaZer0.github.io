---
title: "Memory Management"
author: A1phaZer0
layout: post
category: Kernel
---

> **Address Space**

Logic Address(Virtual Address): Address descripted by `Segment:Offset` is a logic address.

Linear Address: Base address + Offset.

Physical Address: Actual address on address bus.

```bash
| Virtual Address |-----------------------+
         |                                |
         +->| Segment Descriptor |        |
                      |                   |
                      +->| Base Address | |
                                |         |
                                +---------+
                                          |
                                          v
                                  | Linear Address |
                                          |
                                          +->| Page Dir + Page Table + Offset |
                                                           |
                                                           +-> Physical Address
```

Kernel has a `Global Address Space` that means to each task, kernel always have same virtual space, same linear address, same physical address.

Task has its own `Local Address Space` by having its own `LDT` and `Page Table`.

<!--more-->

> **Descriptor**

**Data Segment Descriptor:**
```bash
31                      19    16  14  12         7             0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
|BASE ADDRESS   | | | |A|LIMIT  | |   | | TYPE  |BASE ADDRESS   |
|31~24          |G|B|0|V|19~16  |P|DPL|S|       |23~16          | HIGH
|               | | | |L|       | |   | |0|E|W|A|               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                 | |   |         |  |  | | | | |
                 | |   |         |  |  | | | | +--------> Accessed
                 | |   |         |  |  | | | +----------> Writable
                 | |   |         |  |  | | +------------> Extend Direction 
                 | |   |         |  |  | +--------------> 0 for DATA
                 | |   |         |  |  +----------------> Descriptor Type
                 | |   |         |  |                     1 for CODE/DATA
                 | |   |         |  +-------------------> Descriptor Privilege
                 | |   |         |                        Level (0~3)
                 | |   |         +----------------------> Present
                 | |   +--------------------------------> Available bit
                 | +------------------------------------> 32 bits if set
                 |                                        16 bits if clear
                 +--------------------------------------> Granularity
                                                          4KB/Byte for LIMIT

31                            16 15                            0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
|                               |                               |
|BASE ADDRESS 15~0              |LIMIT 15~0                     | LOW
|                               |                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Code Segment Descriptor:**

```bash
31                      19    16  14  12         7             0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
|BASE ADDRESS   | | | |A|LIMIT  | |   | | TYPE  |BASE ADDRESS   |
|31~24          |G|B|0|V|19~16  |P|DPL|S|       |23~16          | HIGH
|               | | | |L|       | |   | |1|C|R|A|               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                                         | | | |
                                         | | | +--------> Accessed
                                         | | +----------> Readable
                                         | +------------> Conforming
                                         +--------------> 1 for CODE

31                            16 15                            0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
|                               |                               |
|BASE ADDRESS 15~0              |LIMIT 15~0                     | LOW
|                               |                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**System Segment Descriptor:**

```bash
31                      19    16  14  12         7             0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
|BASE ADDRESS   | | | | |LIMIT  | |   | | TYPE  |BASE ADDRESS   |
|31~24          |G| |0| |19~16  |P|DPL|S|       |23~16          | HIGH
|               | | | | |       | |   | |       |               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                                       |
                                       +----------------> 0 for SYSTEM

31                            16 15                            0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
|                               |                               |
|BASE ADDRESS 15~0              |LIMIT 15~0                     | LOW
|                               |                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

`System Segment Descriptor` has two types: **_System Descriptor_** (LDT, TSS) and **_Gate Descriptor_** (Call Gate, Task Gate, Interrupt Gate, Trap Gate).

|TYPE CODE|TYPE
|:---|:---
|0x0|-reserved
|0x1|Available 286 TSS
|0x2|LDT
|0x3|Busy 286 TSS
|0x4|Call Gate
|0x5|Task Gate
|0x6|286 Interrupt Gate
|0x7|286 Trap Gate
|0x8|-reserved
|0x9|Available 386 TSS
|0xA|-reserved
|0xB|Busy 386 TSS
|0xC|386 Call Gate
|0xD|-reserved
|0xE|386 Interrupt Gate
|0xF|386 Trap Gate

> **Paging**

When paging is enabled, the mapping from Linear Address to Physical Address needs two steps.

Step 1. Get page table via page directory.

Step 2. Get page frame via page table.

**Note:** **Page Frame** is one 4K long physical chunk in memory, on the other hand **Page** is 4K long content could be placed in whatever Page Frame.

```bash
Linear Address
31       22 21      12 11         0
+----------+----------+------------+
|  INDEX   |   INDEX  |            |
+----------+----------+------------+
     |           |           |
     |           |           +--------------------------+---> Physical Address
     |           +------------+                         |
     |                        |                         |
     |       Page Directory   |                         |
     |      +--------------+  |                         |
     | +--->|              |  |                         |
     +-|--->|//////////////|--|-+                       |
       |    |              |  | |                       |
       |    |              |  | |        Page Table     |
       |    +--------------+  | |     +--------------+  |
       |                      | +---->|              |  |
       |                      |       |              |  |
      CR3                     +------>|//////////////|--+
                                      |              |
                                      +--------------+
```

Both Page Directory and Page Table contain **Entry** which has the **physical** base address of the corresponding Page Frame.

```bash
Page Directory / Page Table Entry (P=1)
31                                      12                       0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                         |     |   | | |   |U|R| |
|    PAGE FRAME BASE ADDRESS (PHYSICAL)   | AVL |0 0|D|A|0 0|/|/|P|
|                                         |     |   | | |   |S|W| |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                                                     | |     | | |
                                                     | |     | | +--> Present
                                                     | |     | +----> Readable
                                                     | |     |        Writable
                                                     | |     +------> User or
                                                     | |              Superuser
                                                     | |              can access
                                                     | +------------> Accessed
                                                     +--------------> Dirty
                                                     
Page Directory / Page Table Entry (P=0)
31                                                               0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               | |
|                                                               |P|
|                                                               | |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

> **Protection**

To make sure system do not end up with chaos, privilege check is inevitable.

**CPL:** **C**urrent **P**rivilege **L**evel is bit 1, bit 0 in the code selector(cs). This is the only way to know on which privilege level current code is running.

**RPL:** **R**equested **P**rivilege **L**evel is bit 1, bit 0 in other selectors(ds, es, etc.) or selector which is being loaded. RPL can make sure that kernel doesn't access segment on behalf of user when the user doesn't have privilege.

**DPL:** **D**escripor **P**rivilege **L**evel is located in a descriptor. It represents privilege level of segment/gate described.

**Code Segment Access:**

When access a conforming code segment, `CPL >= DPL` is a must. CPL won't be changed.

When access a non-conforming code segment, `CPL == DPL` is a must. CPL won't be changed also.

**Data Segment Access:**

When access a data segment, `MAX(CPL, RPL) <= DPL`.

**System Segment Access:**

**Call Gate**

```bash
31                            16  14  12         7             0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
|                               | |   | | TYPE  |     |PARAM    |
|OFFSET IN SEGMENT 31~16        |P|DPL|S|       |     |COUNT    | HIGH
|                               | |   | |1|1|0|0|     |         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

31                            16 15                            0
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
|                               |                               |
|SEGMENT SELECTOR               |OFFSET IN SEGMENT 15~0         | LOW
|                               |                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

Control transfer via call gate:
1. CPL <= Call Gate DPL; RPL <= Call Gate DPL.
2. For `call`, CPL >= Code Segment DPL no matter what type Code Segment is; For `jmp`, CPL >= Code Segment DPL if it's a `conforming` segment otherwise CPL == Code Segment DPL.

**Stack Switching**

Higher privilege level(0, 1, 2) has its' own stack, Stack Selector and Offset for each stack is located in **TSS** initialized by system. When control is transfered to higher privilege level, new stack will be created by information from **TSS**, some stuff will be copied to new stack.

**NOTE:** No privilege level switch, no stack switch. Hence there is no stack information for privilege level 3 in **TSS**.

```bash
+--------+--------+                  
|        |orig SS | high
+--------+--------+
|    orig ESP     |
+-----------------+<---------------+-----------------+
|     param 1     |                |     param 1     |
+-----------------+                +-----------------+
|     param 2     |                |     param 2     |
+-----------------+                +-----------------+
\                 \                \                 \
/                 /                /                 /
+-----------------+                +-----------------+
|     param N     |                |     param N     |<- orig ESP
+--------+--------+<---------------+-----------------+
|        |orig CS |                |                 |
+--------+--------+                +                 +
|    orig EIP     |                |                 |
+-----------------+                +-----------------+
|                 |
+                 +
|                 |
+                 +
|                 |
+-----------------+
```



http://www.logix.cz/michal/doc/i386/chp06-03.htm
https://manybutfinite.com/post/cpu-rings-privilege-and-protection/
