---
title: "C structure padding and packing"
author: A1phaZer0
layout: post
category: Hacks
---
>**Natural boundary**  

`Natural boundary` is a intel term, to 32 bits arch, it's 4 bytes long since pointer is `4 bytes`; to 64 bits arch, it's `8 bytes` long.  

| arch | 8 bits       | 16 bits | 32 bits | 64 bits | 80 bits | 128 bits |
|:----:|:------------:|:-------:|:-------:|:-------:|:-------:|:--------:|
| x86  |no alignment  |2-byte   |4-byte   |4-byte   |4-byte   |16-byte   |
|x86_64|no alignment  |2-byte   |4-byte   |8-byte   |16-byte  |16-byte   |

<!--more-->

>**Padding**  

* _Basic Type_  

There will be some sort of difference between different data order.  
```cpp
/* case 1 */
int a;
char c;

/* case 2 */
char c;
/* char pad[X]; */
int a;
```
In case 1, there's no padding between `int a` and `char c`, since `int a` is self-aligned to 4-byte (32 bits arch) boundary and `char c` can be aligned anywhere, so right just behind `int a` is a good option to `char c`.  
In case 2, `char c` is placed to any memory address, maybe it's located at the beginning of a 4-byte word, or the end of a word, this situation makes padding between `char c` and `int a` unpredictable.  

* _Structure_  

A struct instance has the alignment as its widest scalar member. This makes struct alignment predictable even char is its first member.  

A struct has trailing padding to make sure if there is a array of this type, next element(struct instance) has the same alignment.  

>**Packing**  

There is a way in gcc, `__attribute__((__packed__))`, to explicitly force packing.  
