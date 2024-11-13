---
title: "Socket Programming - Advanced Techniques"

categories:
    - socket

tags:
    - [Socket Programming, c++, socket, windows, blocking, poll, serialization]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-13
---

# Socket Programming - Chapter 7

> 이 포스트는 `Beej's Guide to Network Programming`을 바탕으로 작성되었습니다.


## Slightly Advanced Techniques

### Blocking
**Block** is a technical terminology for **sleep**
- given the example of `listener`,  it called `recvfrom()`
    * and because there was no data, `recvfrom()` is said to **block** (that is, **sleep** there) until some data arrives
- various functions block
    * `accept()` blocks
    * all the `recv()` functions block

### `poll()`
`poll()` is a function utilized on **Unix-like** systems 
- unfortunately, **Windows** doesn't have a direct equivalent to the Unix `poll()` function
- hence, so you'll need to use one of these **alternatives** based on your requirements
    * `select()`
        * similar to `poll()`
        * suitable to monitor multiple sockets for changes in their status
    * `WSAEventSelect()`
        * suitable for event-driven model
    * Windows I/O Completion Ports (`IOCP`)
        + suitable for a high-performance with large numbers of sockets

### `select()`
`select()` gives you the power to **monitor several sockets** at the **same time**
```c++
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, timeval *timeout);
```
- it'll tell you
    * which ones are ready for **reading**,
    * which are ready for **writing**,
    * and which sockets have raised **exceptions**
- technically speaking, `nfds` parameter should contain `highest file descriptor value + 1`
    * but in **Windows**, `nfds` can have `0` (`stdin`) regardless of the number of sockets in the `fd_set`
        + but it's worth noting that **Windows** does not always support `select()` with `stdin`, even though **Unix-like** systems do
    * the function will **internally figure out** the highest socket number that needs to be checked based on the contents of the `fd_set`
- `readfds`, `writefds`, `exceptfds` point to the **sets** which contains the **sockets** that you want to monitor 
    ```c++
    // these are the macros you need to use for fd_set
    FD_SET(SOCKET fd, fd_set *set);         // Add fd to the set.
    FD_CLR(SOCKET fd, fd_set *set);         // Remove fd from the set.
    FD_ISSET(SOCKET fd, fd_set *set);       // Return true if fd is in the set.
    FD_ZERO(fd_set *set);                   // Clear all entries from the set.
    ```
    * if you want to include standard input stream or output stream, add `0` to the set
    * if you don't care about waiting for a certain set, you can just set it to `NULL` in the call to select()
- `timeout` points to the `timeval` structure which contains the **timeout** information
    * `tv_sec` represents the **seconds** portion of the timeout
    * `tv_usec` represents the **microseconds** portion of the timeout
    * the actual timeout is the combination of both values in **microseconds**
        + **timeout** = `tv_sec` x 1,000,000 + `tv_usec`
    * if you set the fields to `0`, `select()` will timeout **immediately**
    * if you set the parameter timeout to `NULL`, it will **never timeout**
- `select()` returns
    * `SOCKET_ERROR` on error
    * `0` on timeout
- below shows the example of chat program

### chat program - server
```c++

```

### chat program - client
```c++
#include <winsock2.h>
#include <iostream>
#include <ws2tcpip.h>
#include <conio.h>          // For kbhit() to detect keyboard input


#define PORT "9034"         // Port number for communication
#define SERVER "::1"        // IPv6 localhost address (::1) for testing

int main() {
    // Initialize Winsock
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        std::cerr << "WSAStartup failed!" << std::endl;
        return 1;
    }

    // Define the sockaddr_in6 structure for IPv6 server connection
    addrinfo hints, * res;
    memset(&hints, 0, sizeof hints);
    hints.ai_family = AF_INET6;             // Use IPv6
    hints.ai_socktype = SOCK_STREAM;        // TCP socket

    // Get address info for the server using IPv6
    int status;
    if ((status = getaddrinfo(SERVER, PORT, &hints, &res)) != 0) {
        std::cerr << "getaddrinfo: " << gai_strerror(status) << std::endl;
        return 1;
    }

    // Create socket
    SOCKET sock;
    if ((sock = socket(res->ai_family, res->ai_socktype, res->ai_protocol)) == INVALID_SOCKET) {
        std::cerr << "socket failed!" << std::endl;
        return 1;
    }

    // Connect to the server
    if (connect(sock, res->ai_addr, res->ai_addrlen) == -1) {
        std::cerr << "connect failed!" << std::endl;
        closesocket(sock);
        return 1;
    }
    freeaddrinfo(res);  // Done with the address info


    // Set up fd_sets for select
    fd_set setRead;
    fd_set setWrite;
    timeval timeout;

    constexpr unsigned BUF_SIZE = 256;
    char buf[BUF_SIZE];
    int numBytes;

    // Main loop
    while (true) {
        FD_ZERO(&setRead);
        FD_ZERO(&setWrite);

        FD_SET(sock, &setRead);  // We only need to check for input on the socket

        timeout.tv_sec = 2;
        timeout.tv_usec = 0;

        // Wait for data to read from the socket or for user input
        status = select(sock + 1, &setRead, NULL, NULL, &timeout);
        if (status == SOCKET_ERROR) {
            std::cerr << "select failed!" << std::endl;
            closesocket(sock);
            return 1;
        }

        if (FD_ISSET(sock, &setRead)) {
            // We can read data from the socket (server broadcast)
            numBytes = recv(sock, buf, BUF_SIZE, 0);
            if (numBytes <= 0) {
                if (numBytes == 0) {
                    std::cout << "Server closed the connection." << std::endl;
                }
                else {
                    std::cerr << "recv failed!" << std::endl;
                }
                closesocket(sock);
                return 1;
            }

            // Print the received message from other clients (server broadcast)
            buf[numBytes] = '\0'; // Null-terminate the received data
            std::cout << "Received message: " << buf << std::endl;
        }

        // Check for user input (keyboard) using kbhit()
        if (_kbhit()) {
            // If a key was pressed, get the input
            std::cout << "Enter message: ";
            std::cin.getline(buf, BUF_SIZE);

            // Send the message to the server
            if (send(sock, buf, std::strlen(buf), 0) == SOCKET_ERROR) {
                std::cerr << "send failed!" << std::endl;
                closesocket(sock);
                return 1;
            }
        }
    }

    // Clean up and close the socket
    closesocket(sock);
    WSACleanup();

    return 0;
}
```
- it's worth noting that the book doesn't provide the code for **client**, so I manually created the proper one
- moreover, I did use `_kbhit()` to check user's input on standard input stream
    * because `select()` with `stdin` doesn't work in **Windows**
- you can see the result by following the steps below
    * run **server** first
    * run multiple **client**s
    * send any sentence from a **certain client**
    * then the **other clients** will receive it

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}