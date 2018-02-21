---
title: "Signedness and Overflow"
author: A1phaZer0
layout: post
category: Hacks
---
>**CF and OF**  

Carry flag and overflow flag are not that straight forward in integer arithmetic.  
* Carry flag is influenced by unsigned integer artithmetic.  
* Overflow flag otherwise is only relavant to signed integer arithmetic.  

CF is set when carry out from or borrow into MSB(Most Significant Bit), otherwise CF will be cleared.  
<!--more-->

OF is set when MSB turns out to be something contradictary:  
* two positive integers addition result to be negative.  
* two negative integers addition result to be positive.  

Otherwise, no matter result fit or not fit into register, overflow will not be set.  

>**SAL/SHL SAR/SHR**

SAL(**S**hift **A**rithmetic **L**eft) and SHL(**SH**ift Logical **L**eft) have the same opcode, they are exactly the same instruction.  
SAL/SHL puts MSB to CF and clear LSB.  

SAR shifts bits right keeping MSB intact.  
SHR shifts bits right with MSB cleared.  

>**Integer Overflow**  

0xFFFFFFFF -> 0x00000000  
To signed integer, it's not a overflow, since 0xFFFFFFFF is -1(2's complement).  
To unsigned integer, overflow makes integer cut down to 0.  


