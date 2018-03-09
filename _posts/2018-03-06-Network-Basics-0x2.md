---
title: "Network Basics 0x2 Encapsulation"
author: A1phaZer0
layout: post
category: Network
---

> **Encapsulation**

```bash
                                                        +------+
                                                        | Data |
                                                        +------+
                                           +------------+------+
                                           | app header | Data |
                                           +------------+------+
                              +------------+-------------------+
                  TCP segment | tcp header |  application data |
                              +------------+-------------------+
                  +-----------+------------+-------------------+
      IP datagram | ip header | tcp header |  application data |
                  +-----------+------------+-------------------+
+-----------------+-----------+------------+-------------------+---------------+
| Ethernet header | ip header | tcp header |  application data | Ethernet tail |
+-----------------+-----------+------------+-------------------+---------------+
|<------------------------------Ethernet Frame-------------------------------->|
                  |<--------------46~1500 bytes--------------->|
                         
```

> **Data Link Layer**

Data link layer is reponsible for `Ethernet frame` transportation and `ARP/RARP` request and reply.

Data link layer uses 48 bits (6 bytes) `hardware address` (i.e. MAC) as source and destination address. ARP/RARP maps between IP and MAC.