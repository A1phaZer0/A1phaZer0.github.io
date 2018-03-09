---
title: "Network Basics 0x4 User Datagram Protocol"
author: A1phaZer0
layout: post
category: Network
---

> **UDP header**

Big endian, 0~7bit will be transmited first.
```bash
MSB                            LSB
0                               31
+---------------+----------------+
|   src port    |   dest port    |
+---------------+----------------+
|    UDP len    |  UDP checksum  |
+--------------------------------+
\           data(if any)         \
/                                /
+--------------------------------+
```

> **IP fragmentation**

```bash
|<-------------------------IP datagram---------------------------->|
+-----------+------------+-----------------------------------------+
| IP header | UDP header |                  UDP data               |
+-----------+------------+-----------------------------------------+
                           | sliced to...
                           v
+-----------+------------+-------------+
| IP header | UDP header | data frag 1 |
+-----------+------------+-------------+
+-----------+-------------+
| IP header | data frag 2 |
+-----------+-------------+
+-----------+-------------+
| IP header | data frag 3 |
+-----------+-------------+
```

No UDP header is needed except the first fragment.

Through `tcpdump` we have:
```bash
frag IP id:len@offset
```
IP id is a unique integer given by kernel to identify an IP pactet from one host.
