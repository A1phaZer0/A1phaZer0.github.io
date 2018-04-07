---
title: "Network Basics 0x1 socket"
author: A1phaZer0
layout: post
category: Network
---

> **socket, what it is**

socket focuses on data transmitted at the `session layer`.

socket can be identified by (src IP, src PORT, dest IP, dest PORT). Take this as an example, if we try double `telnet 127.0.0.1 80` at the same time, there will be `2 different port` for each `telnet` connection on the client side, on the server side, new connections will be `accept()'d` to different `sockfd`, so even they both use port `80`, new coming data won't mixed together.

**_session layer_**: focuses on establishing and maintaining connections between network applications.

**_presentation layer_**: responsible for presenting data for applications in their way.

**_application layer_**: tracks needs of applications. 

> **socket type**

1. **_Datagram socket_**, also known as connectionless socket. Datagram socket uses `UDP` and should be bound to the `wild card` address in case of broadcasting.

2. **_Stream socket_**, also referenced as connection-oriented socket. Using `TCP` makes it reliable across the network.

3. **_Raw socket_** allows direct sending and receiving of IP packets without any protocol-specific `transport layer` formatting.


<!--more-->

> **functions**

```c
int socket(int domain, int type, int protocol);
```
* domain: protocol family.
* type: socket type. SOCK_STREAM = 1, SOCK_DGRAM = 2...
* protocol: select a protocol from protocol family.
* returns file descriptor of given socket.

```c
int connect(int sockfd, struct sockaddr *remote_host, socklen_t addr_length);
```
Connects a socket(`sockfd`) to a remote host.

```c
int bind(int sockfd, struct sockaddr *local_addr, socklen_t addr_length);
```
Binds a socket to a local address for incoming connections.

```c
int listen(int sockfd, int backlog_queue_size);
```
Queues incoming connection requests up to `backlog_queue_size`.

```c
int accept(int sockfd, struct sockaddr *remote_host, socklen_t *addr_length);
```
Used with connection-based socket types (SOCK_STREAM, SOCK_SEQ‐PACKET). It extracts the first `connection request` on the queue of pending connections for `sockfd`, creates a new connected socket, and returns a new file descriptor referring to that socket.

```c
int send(int sockfd, void *buffer, size_t n, int flags);
```
Sends n bytes from `buffer` to `sockfd`.

```c
int recv(int sockfd, void *buffer, size_t n, int flags);
```
Receives n bytes from `sockfd` into `buffer`, returns length of bytes received.


> **struct sockaddr**

```c
/* common address type */
struct sockaddr                
{                             
	__SOCKADDR_COMMON (sa_); /* address family */
	char sa_data[14];
};

/* internet address type */
struct sockaddr_in
{
	__SOCKADDR_COMMON (sin_);
	in_port_t sin_port;
	struct in_addr sin_addr;

	unsigned char sin_zero[sizeof (struct sockaddr) -
		__SOCKADDR_COMMON_SIZE -
		sizeof (in_port_t) -
		sizeof (struct in_addr)];
};
```

`sin_zero` is used for padding `struct sockaddr_in` to the size of `struct sockadd`, so typecast inbetween these structure types is possible.
