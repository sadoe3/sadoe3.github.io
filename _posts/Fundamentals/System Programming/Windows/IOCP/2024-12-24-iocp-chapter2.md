---
title: "IOCP : Example"

categories:
    - iocp

tags:
    - [C++, System Programming, Windows, OS, IOCP, I/O Functions, chat]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-12-24
---

# Windows Programming : Chapter 2

## Simple Chat Program
In this post, we'll **revise** a simple chat application which we've done [previously](https://sadoe3.github.io/socket/socket-chapter7/#chat-program---server) using `IOCP`

### `IOContext`
It's quite crucial to understand to use `IOContext` (or you can choose different name like `ClientContext` depending on your current situation) to pass the `user-defined object` to another thread that acutally processes the task
```cpp
struct IOContext {
    IOContext(SOCKET inputSocket, bool inputIsRecv) : clientSocket(inputSocket), isRecv(inputIsRecv), referenceCount(0) {
        // initialize recvContext
        pOverlapped = new OVERLAPPED();
        resetOverlapped();

        pWSABuffer = new WSABUF();
        ZeroMemory(data, BUFFER_SIZE);
        pWSABuffer->buf = data;
        pWSABuffer->len = BUFFER_SIZE;
    }
    ~IOContext() {
        delete pOverlapped;
        delete pWSABuffer;
    }
    void addReference() {
        referenceCount.fetch_add(1, std::memory_order_release);
    }
    void release() {
        referenceCount.fetch_sub(1, std::memory_order_acquire);

        if (referenceCount.load(std::memory_order_relaxed) == 0)
            delete this;
    }
    void resetOverlapped() {
        ZeroMemory(pOverlapped, sizeof(OVERLAPPED));
    }

    SOCKET clientSocket;
    OVERLAPPED* pOverlapped;
    WSABUF* pWSABuffer;
    char data[BUFFER_SIZE];
    bool isRecv;
    std::atomic<int> referenceCount;
};
```
```cpp
// WSARecv()

// allocate IOContext object 
IOContext* ioContext = new IOContext(false);
// when it's created, add reference count so that it becomes one
ioContext->addReference();

// associate this socket to IOCP with the IOContext object
CreateIoCompletionPort(reinterpret_cast<HANDLE>(sock), serverIOCP, reinterpret_cast<ULONG_PTR>(recvContext), 0);

DWORD flags = 0;
// overlapped object must be reset before each call to WSARecv, WSASend, or any other function requiring an OVERLAPPED structure
ioContext->resetOverlapped();
// received data will be stored in data[BUFFER_SIZE] through pWSABuffer
WSARecv(socket, ioContext->pWSABuffer, 1, nullptr, &flags, ioContext->overlapped, nullptr);
```
```cpp
// Worker Thread

DWORD bytesTransferred;
ULONG_PTR completionKey;
LPOVERLAPPED pOverlapped;
// wait until new data is received
BOOL result = GetQueuedCompletionStatus(iocp, &bytesTransferred, &completionKey, &pOverlapped, INFINITE);

// cast the completion key to IOContext
IOContext *myContext = reinterpret_cast<IOContext*>(completionKey);

// process it
if(myContext->isSend)
    std:cout << "this task is from WSASend()" << std::endl;
else
    std:cout << "this task is from WSARecv()" << std::endl;
// after using it, decrement the reference count
myContext->release();
```
- As you can see, you can pass the `IOContext` object to another thread through `Completion Key`
- `isRecv` flag is included to **distinguish** whether the task originated from `WSARecv()` or `WSASend()`
- In this example, two distinct types of `IOContext` are required
    * `receiveContext`
        + This type of context exists for **receiving** the data from a **single** `socket`
        + It is **dynamically allocated** when a connection is **established**
            - in other words, the `socket` is created
        + It is **deallocated** when the connection is **terminated**
    * `sendContext`
        + This type of context exists for **sending** the data to **multiple** `sockets`
        + It is **dynamically allocated** when a thread **first** calls `WSASend()` to send to a `specific socket`
        + It is **deallocated** when the **last** `actual sending` has been **completed**
            - This is why the `reference counting` is employed here
- Additionally, `memory_order_release` is applied for **incrementing**, and `memory_order_acquire` is applied for **decrementing** the `reference count`
    * This guarantees that the `increment` operation **happens before** the `decrement` operation
    * It **prevents** scenarios where an `increment` occurs on `already deallocated memory`
- As you can see throug the comments, the `received data` will be stored in `data[BUFFER_SIZE]` through `pWSABuffer`
- Moreover, `overlapped` needs to be **reset**
    * **before** whenever each call to `WSARecv()`, `WSASend()`, or any other function requiring an `OVERLAPPED` structure
    * or **after** processing the results of the previous `overlapped` operation
        + This is because the `OVERLAPPED` structure may contain `residual data` from **previous operations**, leading to **undefined behavior**
    * You can do so by calling `ZeroMemory(ioContext->overlapped, sizeof(OVERLAPPED));`
- It's quite worth noting that the `completion key` is **determined** when `CreateIoCompletionPort()` is called, **not** when `WSASend()` or other function is invoked
    ```cpp
    delete oldContext;
    // Allocate a new IOContext object
    IOContext* newContext = new IOContext();
    // Re-associate with the IOCP port (if necessary, for example if IOContext is tied to a socket)
    CreateIoCompletionPort(reinterpret_cast<HANDLE>(socket), globalIOCP, reinterpret_cast<ULONG_PTR>(newContext), 0);
    ```
    * Therefore, if you want to **change** the `IOContext` for `WSASend()` or `WSARecv(),` you must **reassign** it through `CreateIoCompletionPort()` **before** calling `WSASend()` or `WSARecv()`
- When it comes to `WSARecv()`, it's quite crucial to call `GetQueuedCompletionStatus()` **after** `WSARecv()` is called
    * Otherwise, the thread will **not wait**

### Server
```cpp
#pragma comment(lib, "ws2_32.lib")

#include <iostream>
#include <winsock2.h>
#include <ws2tcpip.h>
#include <thread>
#include <atomic>

#include <vector>

#define PORT "9034"
#define BUFFER_SIZE 1024

// IOContext to handle the data used during the communication
struct IOContext {
    IOContext(SOCKET inputSocket, bool inputIsRecv) : clientSocket(inputSocket), isRecv(inputIsRecv), referenceCount(0) {
        // initialize IOContext
        pOverlapped = new OVERLAPPED();
        resetOverlapped();

        pWSABuffer = new WSABUF();
        ZeroMemory(data, BUFFER_SIZE);
        pWSABuffer->buf = data;
        pWSABuffer->len = BUFFER_SIZE;
    }
    ~IOContext() {
        delete pOverlapped;
        delete pWSABuffer;
    }
    void addReference() {
        referenceCount.fetch_add(1, std::memory_order_release);
    }
    void release() {
        referenceCount.fetch_sub(1, std::memory_order_acquire);

        if (referenceCount.load(std::memory_order_relaxed) == 0)
            delete this;
    }
    void resetOverlapped() {
        ZeroMemory(pOverlapped, sizeof(OVERLAPPED));
    }

    SOCKET clientSocket;
    OVERLAPPED* pOverlapped;
    WSABUF* pWSABuffer;
    char data[BUFFER_SIZE];
    bool isRecv;
    std::atomic<int> referenceCount;
};

// IO Completion Port defined at global scope
HANDLE globalIOCP;

// store all sockets at gloabl scope
class GlobalSockets {
    struct ClientSocket {
        ClientSocket(SOCKET inputSocket) : socketRecv(inputSocket), socketSend(SOCKET_ERROR) {}
        SOCKET socketRecv, socketSend;
    };


public:
    // Refactoring needed!!!
    // Try to use different method like this version commented out
    //bool addSocket(SOCKET inputSocket) {
    //    for (auto& currentClientSocket : mySockets) {
    //        if (areSocketsFromSameIPAndPort(inputSocket, currentClientSocket.socketRecv)) {
    //            currentClientSocket.socketSend = inputSocket;
    //            return true;
    //        }
    //    }
    //    mySockets.emplace_back(inputSocket);
    //    return false;
    //}
    // I chose this option for test purpose only
    // Because this version captures the send and receive socket pair if the clients are connected within a certain timeframe
    bool addSocket(SOCKET inputSocket) {
        if (isFlag) {
            mySockets.emplace_back(inputSocket);
            isFlag = false;
        }
        else {
            mySockets.back().socketSend = inputSocket;
            isFlag = true;
        }
        return isFlag;
    }
    bool isFlag = true;




    void removeSocket(SOCKET targetSocket) {
        auto currentSocket = mySockets.begin(), endSocket = mySockets.end();
        for (; currentSocket != endSocket; ++currentSocket) {
            if (currentSocket->socketRecv == targetSocket)
                break;
        }
        if (currentSocket != endSocket)
            mySockets.erase(currentSocket);
    }
    SOCKET getClientSocketRecv(SOCKET inputSocketSend) {
        auto currentSocket = mySockets.begin(), endSocket = mySockets.end();
        for (; currentSocket != endSocket; ++currentSocket) {
            if (currentSocket->socketSend == inputSocketSend)
                return currentSocket->socketRecv;
        }
        return SOCKET_ERROR;
    }
    auto cbegin() {
        return mySockets.cbegin();
    }
    auto cend() {
        return mySockets.cend();
    }

private:
    /*
    bool areSocketsFromSameIPAndPort(SOCKET sockLeft, SOCKET sockRight) {
        sockaddr_in addrLeft, addrRight;
        int addrLen = sizeof(sockaddr_in);

        // Get the remote address and port of the first socket
        if (getpeername(sockLeft, reinterpret_cast<sockaddr*>(&addrLeft), &addrLen) == SOCKET_ERROR) {
            std::cerr << "Error getting peer name for socket 1: " << WSAGetLastError() << std::endl;
            return false;
        }

        // Get the remote address and port of the second socket
        if (getpeername(sockRight, reinterpret_cast<sockaddr*>(&addrRight), &addrLen) == SOCKET_ERROR) {
            std::cerr << "Error getting peer name for socket 2: " << WSAGetLastError() << std::endl;
            return false;
        }

        // Compare both IP addresses and port numbers
        return addrLeft.sin_addr.s_addr == addrRight.sin_addr.s_addr && addrLeft.sin_port == addrRight.sin_port;
    }
    */
    std::vector<ClientSocket> mySockets;
};
GlobalSockets sockets;

void doWorkerThread() {
    DWORD bytesTransferred;
    ULONG_PTR completionKey;
    LPOVERLAPPED pOverlapped;
    DWORD dwFlags = 0;

    IOContext* ioContext = nullptr;

    while (true) {
        // wait until new network task is queued
        bool result = GetQueuedCompletionStatus(globalIOCP, &bytesTransferred, &completionKey, &pOverlapped, INFINITE);
        ioContext = reinterpret_cast<IOContext*>(completionKey);

        // handle disconnection or error
        if (!result || bytesTransferred == 0) {
            if (bytesTransferred == 0)
                std::cout << "Client (" << ioContext->clientSocket << ") is disconnected " << std::endl;
            else
                std::cerr << "Error occurred.\n";

            closesocket(ioContext->clientSocket);
            // remove clientSocket from the gloabl collection
            ioContext->release();
            sockets.removeSocket(ioContext->clientSocket);
            continue;
        }

        // handle only if this task is from WSARecv
        if (ioContext->isRecv) {
            std::cout << "Received from " << ioContext->clientSocket << "\n";

            // Echo the message back
            // Create a separate IOContext object to avoid data corruption
            IOContext* sendContext = new IOContext(sockets.getClientSocketRecv(ioContext->clientSocket), false);
            memcpy(sendContext->data, ioContext->data, bytesTransferred);

            // Prepare for the next read
            // This should be executed after copying and before processing the received data
            // It's crucial to reset OVERLAPPED structure to call WSARecv() again
            ioContext->resetOverlapped();
            WSARecv(ioContext->clientSocket, ioContext->pWSABuffer, 1, nullptr, &dwFlags, ioContext->pOverlapped, nullptr);

            sendContext->pWSABuffer->len = bytesTransferred;
            for(auto currentSocket = sockets.cbegin(), endSocket = sockets.cend(); currentSocket != endSocket; ++currentSocket) {
                // it's worth noting that ioContext is based on server's perspective, meaning that its socket is for WSARecv() of the server
                // but currentSocket is based on client's perspective
                if (currentSocket->socketSend != ioContext->clientSocket) {
                    // Unlike standard send(), the system will handle the sitaution where the first send() doesn't sends whole data
                    // which means that you don't need to implement a loop for this issue
                    
                    // increase the reference count whenever sendContext is passed to other socket
                    sendContext->addReference();
                    // because you need to use the same overlapped structure for other sockets, you don't need to need reset it in this case
                    // the overlapped object of sendContext is reset through the constructor of sendContext
                    // associate the client's receiving with the IOCP
                    CreateIoCompletionPort(reinterpret_cast<HANDLE>(currentSocket->socketRecv), globalIOCP, reinterpret_cast<ULONG_PTR>(sendContext), 0);
                    WSASend(currentSocket->socketRecv, sendContext->pWSABuffer, 1, nullptr, 0, sendContext->pOverlapped, nullptr);
                }
            }
        }
        else {
            // decrement the reference count after the actual sending is completed
            std::cout << ioContext->data << " sent to " << ioContext->clientSocket << std::endl;
            ioContext->release();
        }
    }
}

int main() {
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        std::cerr << "WSAStartup failed!" << std::endl;
        return 1;
    }

    addrinfo hints, *results, *currentNode;
    int status;
    memset(&hints, 0, sizeof hints);
    hints.ai_family = AF_INET;          
    hints.ai_socktype = SOCK_STREAM;    
    hints.ai_flags = AI_PASSIVE;        

    if ((status = getaddrinfo(NULL, PORT, &hints, &results)) != 0) {
        std::cerr << "server failed: " << gai_strerror(status) << std::endl;
        return 1;
    }

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
    freeaddrinfo(results);    

    if (listen(listeningSocket, 10) == SOCKET_ERROR) {
        std::cerr << "server: failed to listen" << std::endl;
        return 1;
    }

    // New Code: create IOCP
    globalIOCP = CreateIoCompletionPort(INVALID_HANDLE_VALUE, nullptr, 0, 0);

    
    // New Code: initiate worker threads
    std::vector<std::thread> threads;
    constexpr unsigned THREAD_COUNT = 4;
    for (int i = 0; i < THREAD_COUNT; ++i)
        threads.emplace_back(doWorkerThread);
 
    DWORD dwFlags = 0;
    IOContext* ioContext = nullptr;
    SOCKET clientSocket = SOCKET_ERROR;
    while (true) {
        // New Code : block until new connection appears
        clientSocket = accept(listeningSocket, nullptr, nullptr);

        std::cout << "Client connected : " << clientSocket << std::endl;
        // New Code: Create recvContext for this client
        
        if (sockets.addSocket(clientSocket)) {             // add this socket to the global collection
            // socket for client's sending
            // so server needs to receive that data
            ioContext = new IOContext(clientSocket, true);                
            ioContext->addReference();
            // New Code: Associate the client's sending socket with the IOCP
            CreateIoCompletionPort(reinterpret_cast<HANDLE>(clientSocket), globalIOCP, reinterpret_cast<ULONG_PTR>(ioContext), 0);

            // New Code: call WSARecv on the client's sending socket
            WSARecv(clientSocket, ioContext->pWSABuffer, 1, nullptr, &dwFlags, ioContext->pOverlapped, nullptr);
        }
    }

    // clean up
    for (auto& thread : threads)
        thread.join();

    closesocket(listeningSocket);
    CloseHandle(globalIOCP);
    WSACleanup();

    return 0;
}
```
- It's important to note that the program requires **two** `sockets` to communicate with other processes
    * One for **receiving**
    * The other for **sending**
- Otherwise, it could lead to `data corruption`

### Client
```cpp
#pragma comment(lib, "ws2_32.lib")

#include <winsock2.h>
#include <ws2tcpip.h>
#include <thread>
#include <iostream>
#include <vector>

#include <conio.h>          // For kbhit() to detect keyboard input

#define PORT "9034"               // Port number for communication
#define SERVER "127.0.0.1"        // IPv4 loopback address for testing
#define BUFFER_SIZE 1024

// similar to IOContext from Server code, but customized to Client
struct ClientContext {
    ClientContext(SOCKET inputSocket, bool inputIsRecv) : serverSocket(inputSocket), isRecv(inputIsRecv) {
        // initialize recvContext
        pOverlapped = new OVERLAPPED();
        ZeroMemory(pOverlapped, sizeof(OVERLAPPED));

        pWSABuffer = new WSABUF();
        ZeroMemory(data, BUFFER_SIZE);
        pWSABuffer->buf = data;
        pWSABuffer->len = BUFFER_SIZE;
    }
    ~ClientContext() {
        delete pOverlapped;
        delete pWSABuffer;
    }
    void reset() {
        ZeroMemory(pOverlapped, sizeof(OVERLAPPED));
        ZeroMemory(data, BUFFER_SIZE);
        pWSABuffer->buf = data;
        pWSABuffer->len = BUFFER_SIZE;
    }

    SOCKET serverSocket;
    OVERLAPPED* pOverlapped;
    WSABUF* pWSABuffer;
    char data[BUFFER_SIZE];
    bool isRecv;
};

void doWorkerThread(HANDLE serverIOCP) {
    DWORD bytesTransferred;
    ULONG_PTR completionKey;
    LPOVERLAPPED pOverlapped;
    DWORD dwFlags = 0;

    ClientContext* clientContext= nullptr;

    while (true) {
        // wait until new network task is queued
        bool result = GetQueuedCompletionStatus(serverIOCP, &bytesTransferred, &completionKey, &pOverlapped, INFINITE);
        clientContext = reinterpret_cast<ClientContext*>(completionKey);

        // handle disconnection or error
        if (!result || bytesTransferred == 0) {
            if (bytesTransferred == 0)
                std::cout << "Server is disconnected " << std::endl;
            else
                std::cerr << "Error occurred.\n";

            delete clientContext;
            return;
        }

        if (clientContext->isRecv) {
            // handle the received data
            std::cout << "Received: " << clientContext->data<< "\n";

            clientContext->reset();
            WSARecv(clientContext->serverSocket, clientContext->pWSABuffer, 1, 0, &dwFlags, clientContext->pOverlapped, nullptr);
        }
        else {
            // delete the sendContext after the acutal sending is completed
            delete clientContext;
        }
    }
}


int main() {
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        std::cerr << "WSAStartup failed!" << std::endl;
        return 1;
    }

    addrinfo hints, *res;
    memset(&hints, 0, sizeof hints);
    hints.ai_family = AF_INET;
    hints.ai_socktype = SOCK_STREAM;

    int status;
    if ((status = getaddrinfo(SERVER, PORT, &hints, &res)) != 0) {
        std::cerr << "getaddrinfo: " << gai_strerror(status) << std::endl;
        return 1;
    }

    // New Code: two sockets are needed for sending and receiving
    SOCKET socketRecv, socketSend;
    if ((socketRecv = socket(res->ai_family, res->ai_socktype, res->ai_protocol)) == INVALID_SOCKET) {
        std::cerr << "failed to create socket!" << std::endl;
        return 1;
    }
    if ((socketSend = socket(res->ai_family, res->ai_socktype, res->ai_protocol)) == INVALID_SOCKET) {
        std::cerr << "failed to create socket!" << std::endl;
        return 1;
    }

    if (connect(socketRecv, res->ai_addr, res->ai_addrlen) == -1) {
        std::cerr << "failed to connect!" << std::endl;
        closesocket(socketRecv);
        closesocket(socketSend);
        return 1;
    }
    if (connect(socketSend, res->ai_addr, res->ai_addrlen) == -1) {
        std::cerr << "failed to connect!" << std::endl;
        closesocket(socketRecv);
        closesocket(socketSend);
        return 1;
    }
    freeaddrinfo(res);
    std::cout << "Connected to server. Type messages below\n";


    // same code as Server
    HANDLE serverIOCP = CreateIoCompletionPort(INVALID_HANDLE_VALUE, nullptr, 0, 0);

    // asociate the socket for receiving with the IOCP
    ClientContext* recvContext = new ClientContext(socketRecv, true);
    CreateIoCompletionPort(reinterpret_cast<HANDLE>(socketRecv), serverIOCP, reinterpret_cast<ULONG_PTR>(recvContext), 0);

    DWORD dwFlags = 0;
    WSARecv(socketRecv, recvContext->pWSABuffer, 1, nullptr, &dwFlags, recvContext->pOverlapped, nullptr);

    std::vector<std::thread> workerThreads;
    constexpr unsigned THREAD_COUNT = 3;
    for (int i = 0; i < THREAD_COUNT; ++i)
        workerThreads.emplace_back(doWorkerThread, serverIOCP);

    while (true) {
        if (_kbhit()) {
            std::cout << "Enter message: ";
            ClientContext* sendContext = new ClientContext(socketSend, false);
            CreateIoCompletionPort(reinterpret_cast<HANDLE>(socketSend), serverIOCP, reinterpret_cast<ULONG_PTR>(sendContext), 0);
            std::cin.getline(sendContext->data, BUFFER_SIZE);
            sendContext->pWSABuffer->len = std::strlen(sendContext->data);
            if (sendContext->pWSABuffer->len == 0) {
                delete sendContext;
                continue;
            }

            // Send the message to the server
            if (WSASend(socketSend, sendContext->pWSABuffer, 1, 0, 0, sendContext->pOverlapped, nullptr) == SOCKET_ERROR) {
                std::cerr << "send failed!" << std::endl;

                closesocket(socketRecv);
                closesocket(socketSend);
                CloseHandle(serverIOCP);
                WSACleanup();
                return -1;
            }
        }
    }

    // clean up
    for (auto& thread : workerThreads)
        thread.join();

    closesocket(socketRecv);
    closesocket(socketSend);
    CloseHandle(serverIOCP);
    WSACleanup();

    return 0;
}
```




[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}