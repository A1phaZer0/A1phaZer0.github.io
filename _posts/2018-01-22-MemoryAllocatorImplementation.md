---
title: "Memory Allocator Implementation"
author: A1phaZer0
layout: post
category: Kernel
---

> **Memory allocation after booting**

Arrange all pages behind the end of kernel into freelist (memory pool).
```bash
+---------------+<- physical end of memory (if not higher than 2GB)
|               |<- freelist
+---------------+--+
|               |<-+
+---------------+--+
\               \<-+
/               /
+---------------+--+
|               |<-+
+---------------+<- round up to page boundary
|               |
|               |
|               |<- end of kernel
|               |
\               \
/               /
|               |
|               |
|               |<- 0x100000 (1M)
+---------------+
|               |
|               |
|               |
+---------------+<- 0x0
```
Part of Linear/Physical addresses mapping has been crafted before core entry. 

<!--more-->

```bash
[0x80000000, 0x80400000) <==> [0x00000000, 0x00400000)
```
Now puts unused pages to freelist for new page directory and page table allocation.

> **High Zone, Normal Zone**

On 32-bit arch, **Linear Address Space** is as big as [0, 4GB), since kernel base is mapped to `0x80000000` that physical memory higher than 2GB won't be mapped into kernel memory space.

```bash
+---------------+<- 4GB
|               |
|               |
|               |
|               |
|   High Zone   |
|               |
|               |
|               |
|               |
+---------------+<- 0x7FFFFFFF(2GB -1)   +---------------+<- 0xFFFFFFFF
|               |                        |               |
|               |                        |               |
|               |                        |               |
|               |                        |               |
|    Low Zone   |<== Identity Mapping ==>| kernel memory |
|               |                        | space         |
|               |                        |               |
|               |                        |               |
|               |                        |               |
+---------------+<- 0x00000000           +---------------+<- 0x80000000
```

