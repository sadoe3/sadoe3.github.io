---
title: "Socket Programming - System Calls"

categories:
    - socket

tags:
    - [Socket Programming, C, socket, windows, system call]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-12
---

# Socket Programming - Chapter 5

> 이 포스트는 `Beej's Guide to Network Programming`을 바탕으로 작성되었습니다.

## System Calls
You can use the **network functionality** by calling **system calls** or **library calls**
- the point is that these functions should be executed in a proper **order**

### `getaddrinfo()`
```c++
int getaddrinfo(const char *node,               // e.g. "www.example.com" or IP
                const char *service,            // e.g. "http" or port number ("80")
                const struct addrinfo *hints,
                struct addrinfo **res);
```
- you give a **host name** or **IP address** to `node` parameter
- you give a **port number** or **service name** to `service` parameter
- `hints` parameter points to the `addrinfo` structure which contains the relevant information
- if `getaddrinfo()` is executed properly, `res` will point to a **linked list** of `addrinfo`s
```c++
// example use
int status;
struct addrinfo hints;
struct addrinfo *servinfo;          // will point to the results

memset(&hints, 0, sizeof hints);    // make sure the struct is empty
hints.ai_family = AF_UNSPEC;        // don't care IPv4 or IPv6
hints.ai_socktype = SOCK_STREAM;    // TCP stream sockets
hints.ai_flags = AI_PASSIVE;        // fill in my IP for me

if ((status = getaddrinfo(NULL, "3460", &hints, &servinfo)) != 0) {
    fprintf(stderr, "getaddrinfo error: %s\n", gai_strerror(status));
    exit(1);
}

// servinfo now points to a linked list of 1 or more struct addrinfos
// ... do everything until you don't need servinfo anymore ....

freeaddrinfo(servinfo);             // free the linked-list
```
- it's worth noting that if you set `hints.ai_flags` to `AI_PASSIVE`
    * this requests `getaddrinfo()` to assign the address of **my local host** to the socket structures
        + so that you can put `NULL` as the first parameter
    * otherwise (which means you don't set `ai_flags`), you need to put a **specific address** in as the first parameter
- `getaddrinfo()` returns `non-zero` value if there's an error
    * you can print the details by calling `gai_strerror()`
- it's crucial to note that once you get the address information, you need to **free** it after using it just like the dynamic allocation by calling `freeaddrinfo()`


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}