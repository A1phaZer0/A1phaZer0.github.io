---
title: "Network Basics 0x3 Internet Protocol"
author: A1phaZer0
layout: post
category: Network
---

> **IP header**

Big endian, 0~7bit will be transmited first.
```bash
MSB                               LSB
0                                   31
+----+----+--------+----------------+
|ver |len |  TOS   |   total len    |
+----+----+--------+---+------------+
|  identification  |flg|  offset    |
+---------+--------+---+------------+
|   TTL   |protocol| header checksum|
+---------+--------+----------------+
|             source IP             |
+-----------------------------------+
|          destination IP           |
+-----------------------------------+
\         options(if exist)         \
/                                   /
+-----------------------------------+
|                                   |
\                                   \
/               data                /
|                                   |
+-----------------------------------+
```

<!--more-->

> **Subnet addressing**

Subnet id comes from spliting original host id.
```bash
+------------+-------------+
| subnet id  |   host id   |
+------------+-------------+
|<----original host id---->|
```
Use subnet mask to determine subnet id and host id.
```bash
IP: 192.168.26.26
original host id: 26.26 (0x1a1a)
MASK: 255.255.255.0 (0xffffff00)
new host id = IP&(~MASK) = 26
```

> **Special IP addresses**

|IP|MUST BE|Description|
|---|:----:|-----|
|0.0.0.0|SRC|Special host during initialization|
|0.X.X.X|SRC|Special host during initialization|
|127.X.X.X|SRC/DST|Loopback address|
|255.255.255.255|DST|Limited broadcast(never forwarded)|
|All bits set in host id|DST|Broadcast to net id|
|All bits set in host id but subnet id(subnetted)|DST|Broadcast to net id, subnet id|
|All bits set in host id also subnet id(subnetted)|DST|Broadcast to net id|

> **IP routing**

`$ netstat -rn` gives us the routing table.

```bash
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
192.168.30.0    0.0.0.0         255.255.255.0   U         0 0          0 eth1
10.71.0.0       0.0.0.0         255.255.0.0     U         0 0          0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 eth1
0.0.0.0         192.168.30.2    0.0.0.0         UG        0 0          0 eth1
```

```bash
+-----------------+-----------+------------+-----------+---------------+
| Ethernet header | IP header | TCP header | User Data | Ethernet Tail |
+-----------------+-----------+------------+-----------+---------------+
                  |<----------------MTU--------------->|
```
**_MTU_**: **M**aximum **T**ransmission **U**nit. Meaning the biggest **IP packet**.
**_MSS_**: **M**aximum TCP **S**egment **S**ize. MSS = MTU - IP header - TCP header.
**_Window_**: Maximum amount of TCP packets receiver can receive, after that sender will be paused. 
**_irtt_**: initial round trip time.
**_Destination_**: Network that packet is headed to.
**_Genmask_**: Subnet mask.
**_Gateway_**: Gateway(router) where the packet will be sent if `destination` and current `sender` are not in the same network.
**_Flags_**: U G H D M.

|Flags|Description|
|:---:|:----------|
|U| This route is up.|
|G|Destination is a Gateway/router.<br>If G is not set, then it's a direct route, both hardware address and IP of this packet are both set to `Destination`.<br>If G is set, then it's a indirect route, IP is set to `Destination` and hardware address is set to `Gateway`.|
|H|Destination is a host, otherwise a network if not set.|


