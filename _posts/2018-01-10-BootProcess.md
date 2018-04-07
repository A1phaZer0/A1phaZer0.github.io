---
title: "Process of Booting"
author: A1phaZer0
layout: post
category: Kernel
---

> **Loading Stage 1**

Master boot record: If last two bytes of the first sector of disk is 0xAA55, then it's a MBR and will be loaded at 0x7c00 by BIOS.

Through MBR, OS could be loaded step by step.

Disk Image:

```bash
+--------------+----------------------------------------------+
|  BootSector  |              Operating System                |
+--------------+----------------------------------------------+
```
**NOTE:**
1. BootSector is pure binary format code.
2. Operating System is one ELF executable which has ELF headers.

<!--more-->

> **Real Mode**

In real mode(16-bit mode), selector registers store the **base** address of memory, thus `%cs:%ip` gives us the **physical** address: `(cs << 4) + ip`.

In real mode, boot sector enable A20, set up GDT then jump into **Protected Mode**.

> **Protected Mode**

In protected mode, selector registers store the **selector** of segment descriptor.

> **Disk Device**

According to [pcmag][PCMAG]: *SATA is the faster serial version of the parallel ATA (PATA) interface. Both SATA and PATA are "integrated drive electronics" (IDE) devices, which means the controller is in the drive, and only a simple circuit is required on the motherboard. Although SATA is an IDE technology, "IDE" refers to PATA (for details, see IDE, AHCI and PATA/SATA specifications). See mSATA and SATA Express.*

**PIO Mode:** Programmed I/O mode should always supported by ATA-compliant drives. One can use x86 I/O instructions to accomplish disk I/O operations.

There are registers for ATA, **command register**, **status register**, **data register**, **error register**.

**LBA:** Logical Block Addressing, each address describes a block.

**28-bit LBA:** Use 28 bit long address for addressing 0~0x**0**FFFFFFF sector, it's 256M sectors (128GB) in total.

Disk access can be done via I/O ports. Here is the port table for master-ATA disk.


|Port Offset|Function|Description|Param. size LBA28/LBA48
|:---:|:---|:---|:---:|
|0x1F0|Data Port|Read/Write PIO data bytes on this port.|16-bit/16-bit
|0x1F1|Features/Error|Information	Usually used for ATAPI devices.|8-bit/16-bit
|0x1F2|Sector Count|Number of sectors to read/write (0 is a special value).|8-bit/16-bit
|0x1F3|Sector Number/LBAlo|This is CHS/LBA28/LBA48 specific.|8-bit/16-bit
|0x1F4|Cylinder Low/LBAmid|Partial Disk Sector address.|8-bit/16-bit
|0x1F5|Cylinder High/LBAhi|Partial Disk Sector address.|8-bit/16-bit
|0x1F6|Drive/Head Port|Used to select a drive and/or head. May supports extra address/flag bits.|8-bit/8-bit
|0x1F7|Command port/Regular Status port|Used to send commands or read the current status.|8-bit/8-bit

Port **0x1F7** on master-ATA gives us the Status Byte.

|Bit|Abbr.|Function|
|:---:|:---|:---|
|0|ERR|Indicates an error occurred. Send a new command to clear it (or nuke it with a Software Reset).
|3|DRQ|Set when the drive has PIO data to transfer, or is ready to accept PIO data.
|4|SRV|Overlapped Mode Service Request.
|5|DF|Drive Fault Error (does not set ERR).
|6|RDY|Bit is clear when drive is spun down, or after an error. Set otherwise.
|7|BSY|Indicates the drive is preparing to send/receive data (wait for it to clear). In case of 'hang' (it never clears), do a software reset. 

Port **0x3F6** on master-ATA is the Device Control Register/Alternate Status. When this port is written to, it represents **Device Control Register**, when it's read from, it represents **Alternate Status** giving us the same byte as **0x1F7** without affecting interrupts.

|Bit|Abbr.|Function
|:---:|:---|:---
|1|nIEN|Set this to stop the current device from sending interrupts.
|2|SRST|Set this to do a "Software Reset" on all ATA drives on a bus, if one is misbehaving.
|7|HOB|Set this to read back the High Order Byte of the last LBA48 value sent to an IO port. 

**Process of Data Transmission (28-bit LBA PIO on Primary Bus):**

1. Send (0xE0 | ((LBA >> 24) & 0x0f)) to port 0x1F6.
2. Send sector count to port 0x1F2.
3. Send (LBA & 0xff) to port 0x1F3.
4. Send ((LBA >> 8) & 0xff) to port 0x1F4.
5. Send ((LBA >> 16) & 0xff) to port 0x1F5.
6. Send "Read Sectors" command `0x20` to port 0x1F7.
7. Wait until device's not busy.
8. Repeatly read data from 0x1F0.


> **Loading Stage 2**

Read kernel image from disk into memory space starting at 0x100000 (1M).

[PCMAG]:https://www.pcmag.com/encyclopedia/term/50811/sata

