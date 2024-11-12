---
title: "Socket Programming - Sockets"

categories:
    - socket

tags:
    - [Socket Programming, c++, socket, windows, tcp, udp]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-11
---

# Socket Programming - Chapter 2

> 이 포스트는 `Beej's Guide to Network Programming`을 바탕으로 작성되었습니다.

## What is a socket?
A **socket** is a **way to communicate** with other programs over a network
- and its implementation differs across operating systems
    * `file descriptors` in Unix-based systems
    * `socket handles` in Windows
- there are various kinds of sockets
    * but this book deals only with the one type of sockets
        + **Internet Sockets**


## Two Types of Internet Sockets
Although there are numerous types of **Internet Sockets**, this book covers regarding only two sockets
- **Stream Sockets**
    * usually called as `SOCK_STREAM`
- **Datagram Sockets**
    * usually called as `SOCK_DGRAM`
- it's worth noting that **Raw Sockets** are also very useful
    * hence it's recommended to look them up later

### Stream Sockets
**Stream Sockets** are **reliable** two-way connected communication streams
- they use a protocol called `TCP` (transmission control protocol)
    * `TCP` makes sure your data arrives **sequentially** and **error-free** 
- `telnet`, `ssh`, and `HTTP` are the typcial examples which use **stream sockets**

### Datagram Sockets
**Datagram Sockets** are somtimes called **connectionless** sockets
- they use a protocol called `UDP` (user datagram protocol)
    * `UDP` doesn't care about the response of what the sender sent
        + which means that if the sender sent the data, then the connection is **done**
        + **regardless** of whether the recipient receives it or not 
    * this mechanism makes `UDP` **less** reliable than `TCP` but **much faster** than `TCP`
- `tftp` and `video` are the typcial examples which use **datagram sockets**
    * note that `tftp` and other similar programs which need a **reliable** connection have their **own protocol** on top of `UDP`
        + the custom protocol makes the recipient **reply** to the packet that he just received (sending back an `ACK` packet)
        + if the sender **doesn't get** the response in like 5 seconds, he will **re-transmit** the packet **until** he finally gets the response
    * for other programs which can be **unreliable**, you just **ignore** the dropped packets or **cleverly compensate** for them

### Data Encapsulation
**Data encapsulation** in networking is the process of **wrapping** `data` with protocol-specific `headers` at each layer of the **OSI** or **TCP/IP** model to enable proper **transmission** and delivery across the network
- the process of **reversing** data encapsulation is called data **decapsulation**
    + tt involves **removing** the `headers` added at each layer of the **OSI** or **TCP/IP** model as the `data` travels up the stack, ultimately reaching the **application layer** in its original form


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}