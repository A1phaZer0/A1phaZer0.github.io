---
title: "Network Basics 0x5 Transmission Control Protocol"
author: A1phaZer0
layout: post
category: Network
---

> **TCP header**

Big endian, 0~7bit will be transmited first.
Typically(without options), TCP header has a 20 bytes size.
```bash
MSB                                                              LSB
0                                                                 31
+--------------------------------+---------------------------------+
|            src port            |            dest port            |
+--------------------------------+---------------------------------+
|                         sequence number                          |
+------------------------------------------------------------------+
|                      acknowledgement number                      |
+------------------------------------------------------------------+
|          |         |U|A|P|R|S|F|                                 |
|header len|reserved |R|C|S|S|Y|I|           window size           |
|          |         |G|K|H|T|N|N|                                 |
+--------------------------------+---------------------------------+
|            checksum            |           urgent pointer        |
+--------------------------------+---------------------------------+
\                             options                              \
/                                                                  /
+------------------------------------------------------------------+
\                              data                                \
/                                                                  /
+------------------------------------------------------------------+
```
TCP is connection-oriented which means two applications need to establish a connection with each other before data exchanging.

There are exactly two end in TCP communication, so both broadcast and multicast are not possible.

**_sequence number_**: first byte in `data` is `ISN` + kth byte sended, then sequence number is `ISN` + k.
**_acknowledgement number_**: ending sequence number + 1.
**_ISN_**: **I**nitial **S**equence **N**umber of sender when this connection established.

<!--more-->

When a TCP connection established, SYN = 1, this packet consumes one `sequence number`. Also, when a TCP connection finished, FIN = 1, and this packet consumes one `sequence number`.

When a packet received successfully, receiver send back a packed with `ACK = 1` and `ackonwledgement number = sequence number of final byte of data + 1` to ask sender send a packet with new `sequence number = acknowledgement number`.

**_urgent pointer_**: An offset, `sequence number + urgent pointer = sequence number of last byte of data`.

**_URG_**: The urgent pointer is valid.
**_ACK_**: The acknowledgment number is valid.
**_PSH_**: The receiver should pass this data to the application as soon as possible.
**_RST_**: Reset the connection.
**_SYN_**: Synchronize sequence numbers to initiate a connection.
**_FIN_**: The sender is finished sending data.

> **TCP connection establishment and termination**

Via `tcpdump`, we'll get snippets like:
```bash
a > b: Flags [.], cksum 0x9d71 (correct), seq 1:1025, ack 1268, win 35504, length 1024
a > b: Flags [P.], cksum 0xa823 (correct), seq 1025:1032, ack 1268, win 35504, length 7
b > a: Flags [.], cksum 0x3c52 (correct), ack 1032, win 33792, length 0
b > a: Flags [P.], cksum 0xfc96 (correct), seq 1268:2536, ack 1032, win 33792, length 1268
a > b: Flags [.], cksum 0x594d (correct), seq 1032:2056, ack 2536, win 38040, length 1024
a > b: Flags [P.], cksum 0xe47d (correct), seq 2056:2064, ack 2536, win 38040, length 8
```

To each packet:
1. seq 1:1025 means the starting sequence number, the starting sequence number `+ length` for this transmission direction (e.g. a > b starts from 1 but b > a starts from 1032).
2. win 35504 tells `receiver` that window size is 35504 bytes.
3. **S**YN/**F**IN/**R**ST/**P**SH is printed if one of them is set. If none of these four flags set, `.` will be showed. ACK and URG are printed specially.
4. [S.] means SYN and ACK both set.

**Three-way handshake**

TCP connection is bidirectional, means when TCP is establishing SYN packet needs to be sent twice (i.e. a to b, and b to a) and also every packet sent needs a ACK back therefore 3 times handshake.

```bash
a > b: SYN
b > a: SYN ACK
a > b: ACK
```

**Four-way handshake**

```bash
a > b: FIN
b > a: ACK
b > a: FIN
a > b: ACK
```

**TCP half-close**

TCP connection can be closed in one direction.

```bash
a > b: FIN
b > a: ACK
```

After a->b connection closed, application on b will receive the EOF.

**RST flag**

Sender send RST if there's some error, when receiver get RST it'll close connection without acknowledgement.

**ACK delay**

ACK may not be sent back immediately, but delayed waiting for other data, so they can be sent together.

**Nagle algorithm**

Before an ACK received, sender no longer can send other data. On the other hand, in this period, send can just put these not-yet send data together, hence they can be sent next time as one packet.

**sliding window**

Window is a chunk of buffer with FIFO algorithm.


**PSH flag**

Sender send PSH to tell receiver to flush all data in the buffer to the receiving process. 

**Congestion Window/Slow Start**

Sender set congestion window according to number of `ACK` received.

**2MSL**

2 * **M**aximum **S**egment **L**ifetime. MSL is the maximum time that a sender waits before re-sending the packet again.

2MSL is the time sender waits before re-sending `FIN`.
