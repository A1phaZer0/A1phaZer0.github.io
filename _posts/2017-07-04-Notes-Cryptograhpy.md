---
title: "Notes-CryptographyOnLinux-0x0"
author: A1phaZer0
layout: post
category: Hacks
---
## Password encryption
On Linux, passwords can be encrypted through various methods, like AES, MD5, DES, I'll take DES as an example. When password encrypted by DES algorithm, it'll be stored in the pattern: "salt+encryptedpassword", where salt is a two-character string.
<!--more-->
```
                     +random salt(AA)
password("abcdef") -------------------> AAMIOu3emYMks(encrypted password)
```
The frist two characters of encrypted password are the salt, so Linux will check password validation through a process:
```
Get encrypted password, P1
           |
           |
           V
      Extract salt
           |
           |
           V
 Use this salt to encrypt 
target password, result in P2
           |
           |
           V
   Check if P1 == P2
```
