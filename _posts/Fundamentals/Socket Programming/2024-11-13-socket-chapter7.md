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

## `select()`
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
        + **timeout** = `tv_sec` x `1,000,000` + `tv_usec`
    * if you set the fields to `0`, `select()` will timeout **immediately**
    * if you set the parameter timeout to `NULL`, it will **never timeout**
- `select()` returns
    * `SOCKET_ERROR` on error
    * `0` on timeout
- below shows the example of chat program

### chat program - server
```c++
#include <iostream>
#include <winsock2.h>
#include <ws2tcpip.h>       // For additional utilities like inet_pton()

#define PORT "9034"         // Port number for communication

int main() {    
    //Initialize Winsock
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        std::cerr << "WSAStartup failed!" << std::endl;
        return 1;
    }

    // Define the sockaddr_in6 structure for IPv6 server connection
    addrinfo hints, *results, *currentNode;
    int status;
    memset(&hints, 0, sizeof hints);
    hints.ai_family = AF_UNSPEC;         
    hints.ai_socktype = SOCK_STREAM;    // TCP socket
    hints.ai_flags = AI_PASSIVE;        // Use local IP Address

    // Get address info for the server using IPv6
    if ((status = getaddrinfo(NULL, PORT, &hints, &results)) != 0) {
        std::cerr << "server: " << gai_strerror(status) << std::endl;
        return 1;
    }

    // Create socket
    SOCKET listeningSocket;
    for (currentNode = results; currentNode != NULL; currentNode = currentNode->ai_next) {
        listeningSocket = socket(currentNode->ai_family, currentNode->ai_socktype, currentNode->ai_protocol);
        if (listeningSocket < 0)
            continue;
    
        if (bind(listeningSocket, currentNode->ai_addr, currentNode->ai_addrlen) < 0) {
            closesocket(listeningSocket);
            continue;
        }
        break;
    }
    if (currentNode == NULL) {
        std::cerr << "server: failed to bind" << std::endl;
        return 1;
    }
    freeaddrinfo(results);      // Done with the address info
    

    // Set up fd_sets for select
    fd_set setMaster;
    fd_set setRead;
    int maxSocket;
    FD_ZERO(&setMaster);
    FD_ZERO(&setRead);

    // Make as listening socket
    if (listen(listeningSocket, 10) == SOCKET_ERROR) {
        std::cerr << "listen" << std::endl;
        return 1;
    }
    // Add the listeningSocket to the master set
    FD_SET(listeningSocket, &setMaster);
    
    // maxSocket keeps track of the biggest socket
    maxSocket = listeningSocket;
    

    auto getAddr = [](sockaddr* socketAddr) -> void* { if (socketAddr->sa_family == AF_INET) return &(reinterpret_cast<sockaddr_in*>(&socketAddr)->sin_addr); else return &(reinterpret_cast<sockaddr_in6*>(&socketAddr)->sin6_addr); };
    SOCKET newSocket;
    sockaddr_storage clientAddr;
    int addrLength = sizeof clientAddr;

    constexpr unsigned BUF_SIZE = 256;
    char buf[BUF_SIZE];
    int numBytes;
    char remoteIP[INET6_ADDRSTRLEN];

    int currentSocket, currentSocketNested;
    // main loop
    while (true) {
        // Copy master set because select() might modify it
        setRead = setMaster;
        if (select(maxSocket + 1, &setRead, NULL, NULL, NULL) == SOCKET_ERROR) {     // Set timeout NULL so that server always waits for any change
            std::cerr << "select" << std::endl;
            return 1;
        }
        // Search for data to read
        for (currentSocket = 0; currentSocket <= maxSocket; currentSocket++) {
            if (FD_ISSET(currentSocket, &setRead)) {
                if (currentSocket == listeningSocket) {
                    // If listening socket still resides in the read set after select(), this means that new connection request is queued
                    newSocket = accept(listeningSocket, reinterpret_cast<sockaddr*>(&clientAddr), &addrLength);
                    if (newSocket == SOCKET_ERROR) 
                        std::cerr << "accept" << std::endl;
                    else {
                        // If the new socket has been created successfully, then add it to master set and change maxSocket if needed
                        FD_SET(newSocket, &setMaster);
                        if (newSocket > maxSocket)
                            maxSocket = newSocket;
                        std::cout << "server: new connection from " << inet_ntop(clientAddr.ss_family, getAddr(reinterpret_cast<sockaddr*>(&clientAddr)), remoteIP, INET6_ADDRSTRLEN) << " on socket " << newSocket << std::endl;
                    }
                }
                else {
                    // Handle data from a client
                    if ((numBytes = recv(currentSocket, buf, BUF_SIZE, 0)) <= 0) {
                        if (numBytes == 0)
                            std::cout << "server: socket " << currentSocket << " is closed" << std::endl;
                        else
                            std::cerr << "recv" << std::endl;

                        // If there's problem or the connection is closed by the peer, let the server not handle this connection anymore
                        closesocket(currentSocket);
                        FD_CLR(currentSocket, &setMaster);
                    }
                    else {
                        for (currentSocketNested = 0; currentSocketNested <= maxSocket; currentSocketNested++) {
                            // Send to everyone!
                            if (FD_ISSET(currentSocketNested, &setMaster)) {
                                // Except the listeningSocket and the sender
                                if (currentSocketNested != listeningSocket && currentSocketNested != currentSocket) {
                                    if (send(currentSocketNested, buf, numBytes, 0) == SOCKET_ERROR)
                                        std::cerr << "send" << std::endl;
                                }
                            }
                        }
                    }
                }
            }
        }
    }
    WSACleanup();

    return 0;
}
```
- it's worth noting that `select()` might **modify** the given `fd_set`s based on whether the sockets deserve to be in the set or not
    * if some sockets don't deserve, then they would be **removed** from the set
    * hence, if you want to keep track of all clients during the infinite loop
        + you need to make a **master set** which has all the sockets
        + and **copy** it then, use the copied version to `select()`

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

## Serialization
Send **text** data across the network is **easy**
- but what happens if you want to send **other types** of data like `int`, `float` or even `class`
- you have a few options to use
    * convert the number into text with a function like `sprintf()`, then send the text. The receiver will parse the text back into a number using a function like `strtol()`
        + this method is easy, but requires **more space** and **more time** for the conversion
    * just send the data **without any modification**, passing a pointer to the data to `send()`
        ```c++
        // sending
        double d = 3490.15926;
        send(s, &d, sizeof d, 0);
        // receiving
        double d;
        recv(s, &d, sizeof d, 0);
        ```
        + this method may work if the peer's achitecture represents **same** `bit representation` and `byte ordering`
        + otherwise, this method is very **dangerous**, and it's usually referred as **not portable**
        + in case you **don't need portability**, this is nice and fast
    * **encode** the number into a `portable binary form`. The receiver will **decode** it
        + this method is called **serialization**
        + there are various ways to use or implement **serialization**
        + and I'll cover regarding **Boost Serialization** for `C++` code

### Boost Serialization
In order to use **Boost**'s libraries, you need to install it
- there are various ways to accomplish this, I choose to use `vcpkg` (Visual Studio's Package Manager)
    1. install `vcpkg`:
        ```c++
        // clone the vcpkg repository:
        git clone https://github.com/microsoft/vcpkg.git
        cd vcpkg
        .\bootstrap-vcpkg.bat
        // this will install the vcpkg package manager.
        
        setx VCPKG_INSTALLATION_TELEMETRY 0             // type this command if you don't want your usage to be shared
        ```
    2. install **Boost** using `vcpkg`:
        ```c++
        // once vcpkg is installed, use the following command to install Boost:
        .\vcpkg install boost
        // this would take long time because it would install whole libraries
        ```
    3. integrate `vcpkg` with **Visual Studio**:
        ```c++
        // to integrate vcpkg with Visual Studio, run the following command from the vcpkg directory:
        .\vcpkg integrate install
        // this will automatically configure Visual Studio to use the libraries installed by vcpkg.
        ```
    4. use **Boost** in **Visual Studio**:
        + from now on **Boost** will be available for your **existing** and **future** projects
        + you can now use **Boost** headers and libraries by simply **including** them in your C++ project, and **Visual Studio** will automatically link them
- after installing it, you can implement a `class` which support **Boost Serialization**
    ```c++
    #include <boost/serialization/serialization.hpp>
    #include <boost/serialization/access.hpp>

    class SimpleData {
    public:
        // declaration order
        int intValue;
        double doubleValue;

        SimpleData() : intValue(0), doubleValue(0.0) {}
        SimpleData(int i, double d) : intValue(i), doubleValue(d) {}

        template <class Archive>
        void serialize(Archive& ar, const unsigned int version) {
            // need to match the order of declaration
            ar& BOOST_SERIALIZATION_NVP(intValue);
            ar& doubleValue;        // omitting BOOST_SERIALIZATION_NVP is fine
        }
    };
    ```
- it's worth noting that `serialize()` writes the data members to a stream in the **order** in which they are **declared in the class**
- it's **okay** to **omit** `BOOST_SERIALIZATION_NVP` (Named Value Pair) for binary data
    * but if you want to care about **human-readable** formats like `XML` or if you want to **explicitly name** your members
    * it's a good idea to include it
- the example use is shown below

### Boost Serialization - Server
```c++
#include <iostream>
#include <winsock2.h>
#include <boost/archive/binary_iarchive.hpp>
#include <boost/serialization/serialization.hpp>
#include <boost/serialization/access.hpp>


class SimpleData {
public:
    // same code
};

// Function to receive the serialized data over the socket
SimpleData receiveSerializedData(SOCKET sock) {
    // Step 1: Receive data from the socket
    constexpr unsigned BUF_SIZE = 1024;
    char buffer[BUF_SIZE];
    int bytesReceived = recv(sock, buffer, BUF_SIZE, 0);
    if (bytesReceived == SOCKET_ERROR) {
        std::cerr << "Failed to receive data." << std::endl;
        throw std::runtime_error("Receive failed");
    }

    // Step 2: Store received data in a string (buffer) and deserialize it
    std::string receivedData(buffer, bytesReceived);
    std::istringstream iss(receivedData);
    boost::archive::binary_iarchive inputArchive(iss);

    SimpleData data;
    inputArchive >> data; 
    return data;
}

int main() {
    // same code

    // Accept a client connection
    fd_set setRead;
    FD_ZERO(&setRead);
    FD_SET(listeningSocket, &setRead);
    select(listeningSocket + 1, &setRead, NULL, NULL, NULL);

    sockaddr_in clientAddr;
    int clientAddrSize = sizeof(clientAddr);
    SOCKET clientSocket = accept(listeningSocket, (sockaddr*)&clientAddr, &clientAddrSize);
    std::cout << "Client connected." << std::endl;

    // Receive and deserialize data from client
    SimpleData receivedData = receiveSerializedData(clientSocket);
    std::cout << "Received data: " << receivedData.intValue << ", " << receivedData.doubleValue << std::endl;

    // Clean up
    closesocket(clientSocket);
    closesocket(listeningSocket);
    WSACleanup();
    return 0;
}
```
- it's worth noting that you need to implement a `receiveSerializedData()` which uses `boost::archive::binary_iarchive` (input archive) object to **deserialize** the received data
    * you need to use **std::ostringstream** to construct `binary_oarchive` object
    * you can deserialize the data by simply using `>>` (input operator)
        * then, you are able to **send** the **text** data which is **serialized** and returned from `oss.str()`

### Boost Serialization - Client
```c++
#include <iostream>
#include <winsock2.h>

#include <sstream> 
#include <boost/archive/binary_oarchive.hpp>
#include <boost/serialization/serialization.hpp>
#include <boost/serialization/access.hpp>

class SimpleData {
public:
    // same code
};

// Function to send the serialized data over the socket
void sendSerializedData(SOCKET sock, const SimpleData& data) {
    // Step 1: Serialize data into a stringstream (in-memory buffer)
    std::ostringstream oss;
    boost::archive::binary_oarchive outputArchive(oss);
    outputArchive << data;  // Serialize the object into the stream

    // Step 2: Convert serialized data into a byte buffer
    std::string serializedData = oss.str();

    // Step 3: Send the serialized data over the socket
    int result = send(sock, serializedData.c_str(), serializedData.size(), 0);
    if (result == SOCKET_ERROR)
        std::cerr << "Failed to send data." << std::endl;
    else
        std::cout << "Sent " << result << " bytes." << std::endl;
}

int main() {
    // same code

    // Connect to the server
    if (connect(clientSocket, res->ai_addr, res->ai_addrlen) == -1) {
        std::cerr << "connect failed!" << std::endl;
        closesocket(clientSocket);
        WSACleanup();
        return 1;
    }

    std::cout << "Connected to server.\n";

    // Create the data to send
    SimpleData dataToSend(42, 3.14);

    // Serialize and send the data
    sendSerializedData(clientSocket, dataToSend);

    // Clean up
    closesocket(clientSocket);
    WSACleanup();
    return 0;
}
```
- it's worth noting that you need to implement `sendSerializedData()` which uses `boost::archive::binary_oarchive` (output archive) object to **serialize** the data to send
    * you need to use **std::ostringstream** to construct `binary_oarchive` object
    * you can serialize the data by simply using `<<` (output operator)
    * then, you are able to **send** the **text** data which is **serialized** and returned from `oss.str()`


## Data Encapsulation
seems like these two topics might need to go up so that they are part of `techniques`

## Broadcast Packets


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}