---
title: "Socket Programming - Client/Server"

categories:
    - socket

tags:
    - [Socket Programming, C, socket, windows, client-server, client, server]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-12
---

# Socket Programming - Chapter 6

> 이 포스트는 `Beej's Guide to Network Programming`을 바탕으로 작성되었습니다.

## TCP Example
The example below shows th **client** and **server** code where client requests a certain string and server sends it

### Server
```c++
#include <iostream>
#include <winsock2.h>
#include <ws2tcpip.h>  // For additional utilities like inet_pton()
#include <thread>


// Function to handle communication with a client
void handleClient(SOCKET newSocket) {
    const char* message = "Hello, world!";

    // Send data to the client
    if (send(newSocket, message, strlen(message), 0) == SOCKET_ERROR)
        std::cerr << "send failed with error: " << WSAGetLastError() << std::endl;

    // Close the client socket after communication is done
    closesocket(newSocket);
}

int main() {
    // Initialize Winsock
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        std::cerr << "WSAStartup failed!" << std::endl;
        return 1;
    }


    addrinfo hints, * results;
    memset(&hints, 0, sizeof(hints));
    hints.ai_family = AF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM;
    hints.ai_flags = AI_PASSIVE;
    int status = getaddrinfo(NULL, "3490", &hints, &results);
    if (status != 0) {
        std::cerr << "getaddrinfo failed: " << gai_strerror(status) << std::endl;
        WSACleanup();
        return 1;
    }


    SOCKET sock;
    auto currentNode = results;
    for (; currentNode != NULL; currentNode = currentNode->ai_next) {
        if ((sock = socket(currentNode->ai_family, currentNode->ai_socktype, currentNode->ai_protocol)) == INVALID_SOCKET) {
            std::cerr << "server: socket " << WSAGetLastError() << std::endl;
            continue;
        }

        if (bind(sock, currentNode->ai_addr, currentNode->ai_addrlen) == SOCKET_ERROR) {
            closesocket(sock);
            std::cerr << "server: bind " << WSAGetLastError() << std::endl;
            continue;
        }
        break;
    }
    freeaddrinfo(results);      // results completed its job
    if (currentNode == NULL) {
        std::cerr << "server: failed to bind " << std::endl;
        return 1;
    }


    if (listen(sock, 10) == SOCKET_ERROR) {
        std::cerr << "server: failed to listen " << WSAGetLastError() << std::endl;
        closesocket(sock);
        WSACleanup();
        return 1;
    }

    std::cout << "server: waiting for connections..." << std::endl;
    sockaddr_storage clientAddr;
    SOCKET newSock;
    int addrSize = sizeof(clientAddr);
    char clientName[INET6_ADDRSTRLEN];
    auto getAddr = [](sockaddr* socketAddr) -> void* { if (socketAddr->sa_family == AF_INET) return &(reinterpret_cast<sockaddr_in*>(&socketAddr)->sin_addr); else return &(reinterpret_cast<sockaddr_in6*>(&socketAddr)->sin6_addr); };
    while (true) {
        // Accept a new incoming connection
        newSock = accept(sock, (sockaddr*)&clientAddr, &addrSize);
        if (newSock == INVALID_SOCKET) {
            std::cerr << "accept failed with error: " << WSAGetLastError() << std::endl;
            continue;  // Try accepting the next connection
        }


        inet_ntop(clientAddr.ss_family, getAddr(reinterpret_cast<sockaddr*>(&clientAddr)), clientName, sizeof clientName);
        std::cout << "server: got connection from " << clientName << std::endl;

        // Create a new thread to handle the client connection
        // need to include <thread> header file
        // handleClient does its job
        std::thread clientThread(handleClient, newSock);
        clientThread.detach();  // Detach the thread to run independently

        std::cout << "client connected, handling in a new thread." << std::endl;
    }



    // Cleanup
    closesocket(sock);
    WSACleanup();

    return 0;
}
```
- it's worth noting that this code is somehow **modified** into **C++** version
    * but the basic logic is based on the book
- the point regarding `fork()` is that `std::thread` can be the alternative to `fork()`
    * `handleClient()` does the real work

### client
```c++
#include <iostream>
#include <winsock2.h>
#include <ws2tcpip.h>  // For additional utilities like inet_pton()

int main(int argc, char* argv[]) {
    if (argc != 2) {
        std::cerr << "usage: client hostname" << std::endl;
        return 1;
    }

    // Initialize Winsock
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        std::cerr << "WSAStartup failed!" << std::endl;
        return 1;
    }


    addrinfo hints, * results;
    memset(&hints, 0, sizeof(hints));
    hints.ai_family = AF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM;
    int status = getaddrinfo(argv[1], "3490", &hints, &results);
    if (status != 0) {
        std::cerr << "getaddrinfo failed: " << gai_strerror(status) << std::endl;
        WSACleanup();
        return 1;
    }


    SOCKET sock;
    auto currentNode = results;
    for (; currentNode != NULL; currentNode = currentNode->ai_next) {
        if ((sock = socket(currentNode->ai_family, currentNode->ai_socktype, currentNode->ai_protocol)) == INVALID_SOCKET) {
            std::cerr << "client: socket " << WSAGetLastError() << std::endl;
            continue;
        }

        if (connect(sock, currentNode->ai_addr, currentNode->ai_addrlen) == SOCKET_ERROR) {
            closesocket(sock);
            std::cerr << "client: connect " << WSAGetLastError() << std::endl;
            continue;
        }
        break;
    }
    if (currentNode == NULL) {
        std::cerr << "client: failed to connect " << std::endl;
        return 1;
    }

    auto getAddr = [](sockaddr* socketAddr) -> void* { if (socketAddr->sa_family == AF_INET) return &(reinterpret_cast<sockaddr_in*>(&socketAddr)->sin_addr); else return &(reinterpret_cast<sockaddr_in6*>(&socketAddr)->sin6_addr); };
    char serverName[INET6_ADDRSTRLEN];
    inet_ntop(currentNode->ai_family, getAddr(reinterpret_cast<sockaddr*>(currentNode->ai_addr)), serverName, sizeof serverName);
    std::cout << "client: connecting to " << serverName << std::endl;
    
    freeaddrinfo(results);              // now results is done

    const unsigned MAX_DATA_SIZE = 100;
    char buf[MAX_DATA_SIZE];
    int numBytes;
    if((numBytes = recv(sock, buf, MAX_DATA_SIZE - 1, 0)) == SOCKET_ERROR) {
        std::cerr << "client: failed to receive " << std::endl;
        return 1;
    }
    
    buf[numBytes] = '\0';
    std::cout << "client: received " << buf << std::endl;


    // Cleanup
    closesocket(sock);
    WSACleanup();

    return 0;
}
```
- you can see the result by following the steps below
    * run `server` code first
    * run `client` code using **Command Prompt** with `::1` as its second argument
    * then you can see the result
- `::1` is **loopback** address of **IPv6** which is a `special IP address` used by a device to communicate with **itself** over the network
    * in **IPv4**, the **loopback** address range is `127.0.0.0` to `127.255.255.255`
    * the most commonly used loopback address is `127.0.0.1`, which refers to **localhost**
- it's worth noting that if you run `client` code **first**
    * you **cannot** request to `server`
    * because it's a `TCP` world
    * **connection** must be **established first** before doing anything


## UDP Example
The example below shows the **talker** and **listener** code where listener sits on a machine waiting for an incoming packet on the certain port and talker sends a packet to that port, on the specified machine, that contains whatever the user enters on the command line

### Listener
```c++
#include <iostream>
#include <winsock2.h>
#include <ws2tcpip.h>  // For additional utilities like inet_pton()


int main() {
    // Initialize Winsock
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        std::cerr << "WSAStartup failed!" << std::endl;
        return 1;
    }


    addrinfo hints, * results;
    memset(&hints, 0, sizeof(hints));
    hints.ai_family = AF_INET6;
    hints.ai_socktype = SOCK_DGRAM;
    hints.ai_flags = AI_PASSIVE;
    int status = getaddrinfo(NULL, "4210", &hints, &results);
    if (status != 0) {
        std::cerr << "getaddrinfo failed: " << gai_strerror(status) << std::endl;
        WSACleanup();
        return 1;
    }

    SOCKET sock;
    auto currentNode = results;
    for (; currentNode != NULL; currentNode = currentNode->ai_next) {
        if ((sock = socket(currentNode->ai_family, currentNode->ai_socktype, currentNode->ai_protocol)) == INVALID_SOCKET) {
            std::cerr << "listener: socket " << WSAGetLastError() << std::endl;
            continue;
        }

        if (bind(sock, currentNode->ai_addr, currentNode->ai_addrlen) == SOCKET_ERROR) {
            closesocket(sock);
            std::cerr << "listener: bind " << WSAGetLastError() << std::endl;
            continue;
        }
        break;
    }
    if (currentNode == NULL) {
        std::cerr << "listener: failed to bind " << std::endl;
        return 1;
    }

    freeaddrinfo(results);      // results completed its job

    std::cout << "listener: waiting to recvfrom()..." << std::endl;
    const unsigned MAX_BUF_SIZE = 100;
    sockaddr_storage clientAddr;
    int addrSize = sizeof(clientAddr), numBytes;
    char talkerName[INET6_ADDRSTRLEN], buf[MAX_BUF_SIZE];
    
    if ((numBytes = recvfrom(sock, buf, MAX_BUF_SIZE - 1, 0, reinterpret_cast<sockaddr*>(&clientAddr), &addrSize)) == SOCKET_ERROR) {
        std::cerr << "listener: recvfrom" << std::endl;
        return 1;
    }

    auto getAddr = [](sockaddr* socketAddr) -> void* { if (socketAddr->sa_family == AF_INET) return &(reinterpret_cast<sockaddr_in*>(&socketAddr)->sin_addr); else return &(reinterpret_cast<sockaddr_in6*>(&socketAddr)->sin6_addr); };
    std::cout << "listener: got packet from " << inet_ntop(clientAddr.ss_family, getAddr(reinterpret_cast<sockaddr*>(&clientAddr)), talkerName, sizeof talkerName) << std::endl;
    std::cout << "listener: packet is " << numBytes << " bytes long" << std::endl;
    buf[numBytes] = '\0';
    std::cout << "listener: packet contains \"" << buf << "\"" << std::endl;


    // Cleanup
    closesocket(sock);
    WSACleanup();

    return 0;
}
```

### Talker
```c++
#include <iostream>
#include <winsock2.h>
#include <ws2tcpip.h>  // For additional utilities like inet_pton()


int main(int argc, char* argv[]) {
    if (argc != 3) {
        std::cerr << "usage: talker hostname message" << std::endl;
        return 1;
    }

    // Initialize Winsock
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        std::cerr << "WSAStartup failed!" << std::endl;
        return 1;
    }


    addrinfo hints, * results;
    memset(&hints, 0, sizeof(hints));
    hints.ai_family = AF_INET6;
    hints.ai_socktype = SOCK_DGRAM;
    int status = getaddrinfo(argv[1], "4210", &hints, &results);
    if (status != 0) {
        std::cerr << "getaddrinfo failed: " << gai_strerror(status) << std::endl;
        WSACleanup();
        return 1;
    }


    SOCKET sock;
    auto currentNode = results;
    for (; currentNode != NULL; currentNode = currentNode->ai_next) {
        if ((sock = socket(currentNode->ai_family, currentNode->ai_socktype, currentNode->ai_protocol)) == INVALID_SOCKET) {
            std::cerr << "talker: socket " << WSAGetLastError() << std::endl;
            continue;
        }

        break;
    }
    if (currentNode == NULL) {
        std::cerr << "talker: failed to create socket" << std::endl;
        return 1;
    }

    int numBytes;
    if ((numBytes = sendto(sock, argv[2], strlen(argv[2]), 0, currentNode->ai_addr, currentNode->ai_addrlen)) == SOCKET_ERROR) {
        std::cerr << "talker: sendto" << std::endl;
        return 1;
    }  
    
    freeaddrinfo(results);              // now results is done


    std::cout << "talker: sent " << numBytes << " bytes to " << argv[1] << std::endl;

    // Cleanup
    closesocket(sock);
    WSACleanup();

    return 0;
}
```
- you can see the result by following the steps below
    * run `listener` code first
    * run `talker` code using **Command Prompt** with `::1` as its second argument, and any `single word` as its third parameter
    * then you can see the result
- the point here is that if you run `talker` **first**
    * it's still **possible** to send a message
    * but that message would be **disappeared**
    * because it's a `UDP` world



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}