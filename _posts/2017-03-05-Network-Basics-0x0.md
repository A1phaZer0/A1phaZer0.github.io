---
title: "Network Basics 0x0 Infrasturcture and terms"
author: A1phaZer0
layout: post
category: Network
---

> **Infrastructure and terms**

**_Twist-pair wiring_**: for the purposes of improving electromagnetic compatibility.

**_Network Interface Controller_**: NIC, connects a computer to computer network. Both a `physical layer` (access a networking medium) and a `data linke layer` (address via MAC) device.

**_8P8C_**: **8 P**osition **8 C**ontact. Commonly refered to as a `Registered Jack 45`(RJ45), but it's incorrect.

**_Berkeley Packet Filter_**: BPF, provides a raw interface to `data link layer`, permitting raw link-layer packets to be sent and received. It can receive all packets if NIC is at `promiscuous mode`. 

**_Network Byte Order_**: it's a big-endian, MSB at lower position.

**_port_**: used for external endpoint of a node.

**_socket_**: used for internal endpoint of a node.

<!--more-->

**_socket decriptor/socket address_**: Internally, process can reference a socket by socket descriptor (like a file descriptor). Externally, one computer can communicate to another via socket address, IP:PORT binding with a specific socket.

**_gateway_**: used for translation between networks using different protocols.

**_demultiplexing_**: peel off header and send packet from bottom to top layer by layer.

> **IP address**

IP: xxx.xxx.xxx.xxx (4 bytes/32 bits total)

```bash
        +-+-------+------------------------+
class A |0| netid |        hostid          |   0.0.0.0 ~ 127.255.255.255
        +-+-------+------------------------+
                  |<--------24bits-------->|
        +--+--------------+----------------+
class B |10|     netid    |      hostid    |   128.0.0.0 ~ 191.255.255.255
        +--+--------------+----------------+
                          |<----16bits---->|
        +---+---------------------+--------+
class C |110|        netid        | hostid |   192.0.0.0 ~ 223.255.255.255
        +---+---------------------+--------+
                                  |< 8bits>|
        +----+-----------------------------+
class D |1110|      multicast group id     |   224.0.0.0 ~ 239.255.255.255
        +----+-----------------------------+
	
        +-----+----------------------------+
class E |11110|                            |   Reserved
        +-----+----------------------------+
```

If a host has multiple NICs then it'll have multiple IPs.

> **Tools**

**ss/netstat**: check established sockets.

**netstat**: -n (use number not symbol), -t (tcp connection) -u (udp) -p (list program using this socket) -l (listen)

**netstat -tlpn**: list all listening sockets using tcp connections and programs using these sockets by a numeric style.

**wireshark**: for network sniffing.

**tcpdump**:
```bash
-XX:   show hex and ascii
-A:    show ascii
-D:    list all device
-i:    device to inspect
-c:    count of packets to dump
-v:    verbose
-n:    numeric
-udp:  for udp
-icmp: for icmp
host:  'host XX:XX:XX:XX' filter for specific host
port:  'port XXXX' filter for specific port
'host 127.0.0.1 && port 22'
'host 127.0.0.1 && (port 22 or port 21)'
'(host 127.0.0.1 || host 192.168.0.1) && (port 22 || port 21)'
```

