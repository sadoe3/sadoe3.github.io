---
title: "Socket Programming - Data Types"

categories:
    - socket

tags:
    - [Socket Programming, C, socket, windows, struct, data types]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-11
---

# Socket Programming - Chapter 3

> 이 포스트는 `Beej's Guide to Network Programming`을 바탕으로 작성되었습니다.

## Data Types used by the Sockets Interface

### `addrinfo`
```c++
struct addrinfo {
    int ai_flags;               // AI_PASSIVE, AI_CANONNAME, etc.
    int ai_family;              // AF_INET, AF_INET6, AF_UNSPEC
    int ai_socktype;            // SOCK_STREAM, SOCK_DGRAM
    int ai_protocol;            // use 0 for "any"
    size_t ai_addrlen;          // size of ai_addr in bytes
    struct sockaddr *ai_addr;   // struct sockaddr_in or _in6
    char *ai_canonname;         // full canonical hostname

    struct addrinfo *ai_next;   // linked list, next node
};
```
- you can get the `pointer` to a new linked list of these structures by calling `getaddrinfo()`
- `ai_family` field sets which version of `IP` to use
    * `AF_INET` for IPv4
    * `AF_INET6` for IPv6
    * `AF_UNSPEC` to use whatever

### `sockaddr`
```c++
struct sockaddr {
    unsigned short sa_family;   // address family, AF_xxx
    char sa_data[14];           // 14 bytes of protocol address
};
```
- `sa_data` contains a **destination** address and **port** number for the socket

### `sockaddr_in`
```c++
// (IPv4 only--see struct sockaddr_in6 for IPv6)
struct sockaddr_in {
    short int sin_family;           // Address family, always AF_INET
    unsigned short int sin_port;    // Port number
    struct in_addr sin_addr;        // Internet address
    unsigned char sin_zero[8];      // Same size as struct sockaddr
};
```
- this structure is used in order to **easily represent** the IP **address**
- it's worth noting that a `pointer` to a `sockaddr_in` cna be **cast** to a `pointer` to a `sockaddr`
- `sin_zero` should be set to all zeros with the function `memset()`
- this structure is for `IPv4`, hence `sin_family` must be set to `AF_INET`
- `sin_port` must be in **Network Byte Order** by using `htons()`

### `in_addr`
```c++
// (IPv4 only--see struct in6_addr for IPv6)
// Internet address (a structure for historical reasons)
struct in_addr {
    uint32_t s_addr;    // that's a 32-bit int (4 bytes)
}
```

### `sockaddr_in6`
```c++
// (IPv6 only)
struct sockaddr_in6 {
    u_int16_t sin6_family;      // address family, AF_INET6
    u_int16_t sin6_port;        // port number, Network Byte Order
    u_int32_t sin6_flowinfo;    // IPv6 flow information
    struct in6_addr sin6_addr;  // IPv6 address
    u_int32_t sin6_scope_id;    // Scope ID
};
```

### `in6_addr`
```c++
// (IPv6 only)
struct in6_addr {
    unsigned char s6_addr[16];  // IPv6 address
};
```

### `sockaddr_storage`
```c++
struct sockaddr_storage {
    sa_family_t ss_family;      // address family
    // all this is padding, implementation specific, ignore it:
    char __ss_pad1[_SS_PAD1SIZE];
    int64_t __ss_align;
    char __ss_pad2[_SS_PAD2SIZE];
};
```
- this structure is **large** enough to hold both `IPv4` and `IPv6` stuctures
- based on the value of `ss_family`, you can **cast** this structure to `sockaddr_in` or `sockaddr_in6`


## Functions regarding them

### `inet_pton()`
```c++
sockaddr_in sa;      // IPv4
sockaddr_in6 sa6;    // IPv6

inet_pton(AF_INET, "11.11.120.17", &(sa.sin_addr));             // IPv4
inet_pton(AF_INET6, "2302:db28:6b3:1::3590", &(sa6.sin6_addr)); // IPv6
```
- it's worth noting that you don't specify `struct` keyword to use structures defined in `C` when you use it in `C++` source file
- `inet_pton()` converts an IP address in numbers-and-dots notation into either `in_addr` or `in6_addr` depending on whether you specify `AF_INET` or `AF_INET6`
    * `pton` stands for **presentation to network**
- `inet_pton()` returns
    * `-1` on **error**
    * `0` if the address is messed up
- hence it's recommended to check to make sure the result is greater than `0` before using

### `inet_ntop()`
```c++
// IPv4
char ip4[INET_ADDRSTRLEN];      // space to hold the IPv4 string
sockaddr_in sa;          // pretend this is loaded with something

inet_ntop(AF_INET, &(sa.sin_addr), ip4, INET_ADDRSTRLEN);
std::cout << "The IPv4 address is: " << ip4 << std::endl;


// IPv6:
char ip6[INET6_ADDRSTRLEN];     // space to hold the IPv6 string
sockaddr_in6 sa6;        // pretend this is loaded with something

inet_ntop(AF_INET6, &(sa6.sin6_addr), ip6, INET6_ADDRSTRLEN);
std::cout << "The IPv6 address is: " << ip6 << std::endl;
```
- `inet_ntop()` does the opposite job of `inet_pton()`
- it's worth noting that `INET_ADDRSTRLEN` and `INET6_ADDRSTRLEN` are **macro**s for representing the **size** of the string
- also these functions (`inet_pton()`, `inet_ntop()`) only work with **numeric IP addresses**
    * which means that they **won’t do** any nameserver **DNS lookup** on a hostname, like `www.example.com`
- `getaddrinfo()` does that 


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}