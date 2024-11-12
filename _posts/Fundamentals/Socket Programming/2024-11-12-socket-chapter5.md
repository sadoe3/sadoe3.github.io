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

### `socket()`
```c++
SOCKET socket(int domain, int type, int protocol);
```
- it's worth noting that the **return type** of `socket()` in **Windows** is `SOCKET` not `int`
- `domain` parameter is `PF_INET` or `PF_INET6`
    * it's worth noting that `PF_INET` (protocal famaily) thing is **closely related** to its `AF` (address family) version
    * but the most **correct** thing to do is to use `AF_INET` in your `sockaddr_in` and `PF_INET` in your call to `socket()`
- `type` parameter is `SOCK_STREAM` or `SOCK_DGRAM`
- `protocol` parameter can be set to `0` to choose the proper protocol for the given type.
    * or you can call `getprotobyname()` to look up the protocol you want like `tcp` 
    ```c++
    // example use
    addrinfo hints, * res;
    memset(&hints, 0, sizeof(hints));
    hints.ai_family = AF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM;
    int status= getaddrinfo("www.example.com", "http", &hints, &res);
    if (status != 0) {
        std::cerr << "getaddrinfo failed: " << gai_strerror(status) << std::endl;
        WSACleanup();
        return 1;
    }

    SOCKET sock = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
    if (sock == INVALID_SOCKET) {
        std::cerr << "Socket creation failed with error: " << WSAGetLastError() << std::endl;
        WSACleanup();
        return 1;
    }


    // Cleanup
    freeaddrinfo(res);
    closesocket(sock);
    ```
- if there's an error, `socket()` returns `INVALID_SOCKET`
    * you can print the details by calling `WSAGetLastError()`
- just like `freeaddrinfo()`, you need to call `closesocket()` after using the socket
    * the proper order of cleanup
        + `freeaddrinfo()` - `closesocket()` - `WSACleanup()`

### `bind()`
```c++
int bind(SOCKET s, sockaddr *name, int namelen);
```
- `s` parameter is the socket returned by `socket()`
- `name` parameter is a pointer to a `sockaddr` structure that contains information about your address
- `namelen` parameter is the length in bytes of that address
    ```c++
    // example use
    addrinfo hints, * res;
    memset(&hints, 0, sizeof(hints));
    hints.ai_family = AF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM;
    hints.ai_flags = AI_PASSIVE;
    int status= getaddrinfo(NULL, "3450", &hints, &res);
    if (status != 0) {
        std::cerr << "getaddrinfo failed: " << gai_strerror(status) << std::endl;
        WSACleanup();
        return 1;
    }


    SOCKET sock = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
    if (sock == INVALID_SOCKET) {
        std::cerr << "socket creation failed with error: " << WSAGetLastError() << std::endl;
        WSACleanup();
        return 1;
    }


    status = bind(sock, res->ai_addr, res->ai_addrlen);
    if (status == SOCKET_ERROR) {
        std::cerr << "bind failed with error: " << WSAGetLastError() << std::endl;
        closesocket(sock);
        WSACleanup();
        return 1;
    }
    ```
- if there's an error, `bind()` returns `SOCKET_ERROR`
- unlike the functions above, you **don't need to call** anything when you're done
- if you get the error code `10049`, then try to use your **locol IP address** because that code means it's impossible to bind
- it's worth noting that if you are `connect()`ing to a remote machine and you **don’t care what your local port** is (as is the case with `telnet` where you only care about the remote port)
    * you can simply call `connect()` **only**
    * it will check to see if the socket is unbound, and will `bind()` it to an unused local port necessary

### `connect()`
```c++
int connect(SOCKET s, sockaddr *name, int namelen);
```
- `connect()` is quite **similar** to `bind()`
    * because they have the same parameters, same return value
- the difference is that
    * `bind()` is related to the **local** address and port number
    * `connect()` is related to the **remote** connection
    ```c++
    // example use
    addrinfo hints, * res;
    memset(&hints, 0, sizeof(hints));
    hints.ai_family = AF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM;
    int status= getaddrinfo("www.example.com", "http", &hints, &res);
    if (status != 0) {
        std::cerr << "getaddrinfo failed: " << gai_strerror(status) << std::endl;
        WSACleanup();
        return 1;
    }
    SOCKET sock = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
    if (sock == INVALID_SOCKET) {
        std::cerr << "socket creation failed with error: " << WSAGetLastError() << std::endl;
        WSACleanup();
        return 1;
    }


    status = connect(sock, res->ai_addr, res->ai_addrlen);
    if (status == SOCKET_ERROR) {
        std::cerr << "connect failed with error: " << WSAGetLastError() << std::endl;
        closesocket(sock);
        WSACleanup();
        return 1;
    }
    ```
- it's worth noting that `bind()` is **not called** because specifying local port is not necessary

### `listen()`
```c++
int listen(SOCKET s, int backlog);
```
- `listen()` puts the data inside its queue of which size is set by the `backlog` paramemter
    * if the sender sends more data, the outnumbered data are handled based on which OS you use
    * some of them may be **refused** or **dropped**
- `s` parameter is the `SOCKET`
- `backlog` parameter sets the **number** of **connections allowed** on the incoming queue
- the example use is below with `accept()`

### `accept()`
```c++
SOCKET accept(SOCKET s, sockaddr *addr, int *addrlen);
```
- `accept()` returns a **new** `SOCKET` which contains the data inside the **queue** which `listen()` handles with
    * the point is that the **original** `SOCKET` is still **listening**
- `s` parameter is the original `SOCKET`
- `addr` parameter will usually be a pointer to a local `sockaddr_storage` structure
    * this is where the information about the incoming connection will go
- `addrlen` is a local integer variable that should be set to `sizeof(sockaddr_storage)` **before** its address its address is passed to `accept()`
    * `accept()` will not put more than that many bytes into `addr`
    * if it puts **fewer** in, it'll **change** the value of `addrlen` to reflect that
    ```c++
    // example use
    addrinfo hints, * res;
    memset(&hints, 0, sizeof(hints));
    hints.ai_family = AF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM;
    hints.ai_flags = AI_PASSIVE;
    int status= getaddrinfo(NULL, "3490", &hints, &res);
    if (status != 0) {
        std::cerr << "getaddrinfo failed: " << gai_strerror(status) << std::endl;
        WSACleanup();
        return 1;
    }
    SOCKET sock = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
    if (sock == INVALID_SOCKET) {
        std::cerr << "socket creation failed with error: " << WSAGetLastError() << std::endl;
        WSACleanup();
        return 1;
    }
    status = bind(sock, res->ai_addr, res->ai_addrlen);
    if (status == SOCKET_ERROR) {
        std::cerr << "bind failed with error: " << WSAGetLastError() << std::endl;
        closesocket(sock);
        WSACleanup();
        return 1;
    }



    status = listen(sock, 5);
    if (status == SOCKET_ERROR) {
        std::cerr << "listen failed with error: " << WSAGetLastError() << std::endl;
        closesocket(sock);
        WSACleanup();
        return 1;
    }

    sockaddr_storage theirAddr;
    int addrSize = sizeof theirAddr;
    SOCKET newSocket = accept(sock, reinterpret_cast<sockaddr*>(&theirAddr), &addrSize);
    if (newSocket == INVALID_SOCKET) {
        std::cerr << "accept failed with error: " << WSAGetLastError() << std::endl;
        closesocket(sock);
        WSACleanup();
        return 1;
    }
    ```

### `send()` 
```c++
int send(SOCKET s, const char *buf, int len, int flags);
// example use
char *msg = "Kyle was here!";
int len, bytes_sent;

len = strlen(msg);
bytes_sent = send(sock, msg, len, 0);
```
- it's worth noting that these 2 functions are related to **stream** sockets or **connected datagram** sockets
    * if you want to use regualr **unconnected** datagram sockets, you need to call `sendto()` or `recvfrom()`
- `s` parameter is the `SOCKET`
- `buf` parameter is the pointer to the data you want to send
- `len` parameter is the length of that data in **bytes**
- `flags` would be covered in later chapter
    * just set it to `0` for now
- `send()` returns the number of bytes sent
    * `-1` means error

### `recv()`
```c++
int recv(SOCKET s, const char *buf, int len, int flags);
```
- `recv()` works similarly to how `send()` works
    * it returns the number of bytes read
        + `-1` means error

### `sendto()` 
```c++
int sendto(SOCKET s, const const *buf, int len, int flags, const sockaddr *to, int tolen);
```
- `to` parameter points to a `sockaddr` structure which contains the **destination** IP address and port
- `tolen` is the size of the structure to which `to` points
    * it can be `sizeof *to`
- the rest is same as `send()`

### `recvfrom()`
```c++
int recvfrom(SOCKET s, const const *buf, int len, int flags, const sockaddr *from, int *fromlen);
```
- `from` parameter points to a local `sockaddr_storage` that will be filled with the IP address and port of the originating machine
- `fromlen` parameter is a pointer to a local `int`
    - and this should be initialized to `sizeof *from` or `sizeof(sockaddr_storage)`
- the rest is same as `recv()`

### `shutdown()`
```c++
int shutdown(SOCKET s, int how);
```
- `shutdown()` will prevent any more reads and writes to the socket
    * but it's worth noting that it **doesn’t actually close** the socket
        + it just changes its usability.
    * in order to **free** a socket, you need to call `close()`
- `how` Effect
    * `0`: Further **receives** are disallowed
    * `1`: Further **sends** are disallowed
    * `2`: Further **sends** and **receives** are disallowed (like close())

### `getpeername()`
```c++
int getpeername(SOCKET s, sockaddr *name, int *namelen);
```
- `getpeername()` will tell you the **information** regarding the **other side** of the connection
- `name` points to a `sockaddr` structure (or a `sockaddr_in`) that will hold the **information**
- `namelen` is a pointer to an `int`
    * which should be initialized to `sizeof *addr` or `sizeof(sockaddr)`
- it returns `-1` on error
- once you have their address
    * you can use `inet_ntop()`, `getnameinfo()`, or `gethostbyaddr()` to print or get more information

### `gethostname()`
```c++
int gethostname(char *hostname, int size);
```
- `gethostname()` will tell you the name of the computer that your program is running on
    * that name can then be used by `getaddrinfo()` to determine the IP address of your local machine
- `hostname` points to an array of chars that will contain the **hostname** upon the function’s return 
- `size` is the length in `bytes` of the hostname array
- it returns `0` on **successful** completion
    * and `-1` on **error**



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}