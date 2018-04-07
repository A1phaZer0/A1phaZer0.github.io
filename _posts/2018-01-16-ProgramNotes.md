---
title: "Notes of Programming"
author: A1phaZer0
layout: post
category: Kernel
---

> **Designated Initializer**

Designated initializer can only be used with `constant` or `pointer`, and this `pointer` is directly provided by linker (e.g. \_etext, \_edata, \_end).
```c
extern char *data;
#define CONSTANT 10

static abc[10] = {
	[0] = CONSTANT,
	[1] = data,
};
```

<!--more-->
