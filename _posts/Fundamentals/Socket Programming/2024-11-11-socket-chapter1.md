---
title: "Socket Programming : Initial Settting"

categories:
    - socket

tags:
    - [Socket Programming, c++, socket, windows]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-11
---

# Socket Programming - Chapter 1

> 이 포스트는 `Beej's Guide to Network Programming`을 바탕으로 작성되었습니다.

## Before Using the Socket Libraries

### Header File
You must include `<winsock2.h>`
- if you include `<windows.h>`, then it automatically includes the **older** version (`winsock.h`) header file
    * in order to fix this confliction, **macro** should be defined to handle this problem
- moreover, you need to tell your compiler to **link** in the Winsock library, called `ws2_32.lib` for Winsock 2
    * if you're using **Visual Studio** then follow the steps below
    * project name -> right click -> properties -> under Configuration Properties -> Linker -> Input -> Additional Dependencies -> add `ws2_32.lib;`
        + then the Additional Dependencies should look like this by default : `ws2_32.lib;%(AdditionalDependencies)`
    * After applying this change, it would work properly
- moreover, you need to include `<ws2tcpip.h>` to use additional functionality related to socket programming 

### Functions
You need to call `WSAStartup()` before handling with the sockets library and call `WSACleanup()` after using them
```c++
// initial settings for socket programming
#include <winsock2.h>
#include <ws2tcpip.h>
#include <iostream>

int main() {
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        std::cerr << "Winsock initialization failed!" << std::endl;
        return 1;
    }

    // some codes

    WSAStartup()
    return 0;
}
```
- it's worth noting that `WSA` stands for **Windows Sockets API** 

### Difference Linux and Windows
- in order to close socket in Windows
    * you need to call `closesocket()` not `close()`
- `select()` only works with **socket descriptors**, not file descriptors in Windows
- there is no `fork()` in Windows
    * you should call `CreateProcess()` or `CreateThread()` which are beyond the scope of the document



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}