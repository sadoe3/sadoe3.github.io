---
title: "IOCP : Basic Understanding"

categories:
    - iocp

tags:
    - [C++, System Programming, Windows, OS, IOCP]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-12-23
---

# IOCP : Chapter 1

> This post is based on [**I/O Completion Ports**](https://learn.microsoft.com/en-us/windows/win32/fileio/i-o-completion-ports).

## I/O Completion Port
`I/O completion ports` (IOCP) offer an efficient **threading mechanism** for handling **multiple** `asynchronous I/O requests`, particularly on multiprocessor systems
- When an `I/O completion port` is created, the system establishes a `dedicated queue` for `threads` designed specifically to handle `these requests`

### Structure of IOCP in a Process
- One `IO Completion Port`
- Multiple `File Handles`
    * A **single** `IOCP` can manage **multiple** `file handles`
    * The term `file handle` as used here refers to a system abstraction representing an **overlapped I/O endpoint**, not only a `file` on disk
        + For example, it can be a `network endpoint`, `TCP socket`, `named pipe`, or `mail slot`
        + Any system object that supports **overlapped I/O** can be used
- Two `Queues`
    * `FIFO Queue` for **Tasks**
        + The IOCP maintains a queue where `completion packets` (representing `tasks` or `I/O results`) are stored in **FIFO** order
        + It follows **FIFO** order
            - the **oldest** task in the queue is processed **first**
        + These tasks can come from either **automatic** or **manual** posting
    * `LIFO Queue` for **Threads**:
        + `Threads` waiting for the `tasks` are maintained in a **LIFO** order
        + It follows **LIFO** order
            - the **most recently waiting** thread is **chosen** to process the task

### How IOCP Works
You can use IOCP with supported `I/O functions` (described in the next section), but it's crucial to first understand the basic workflow
- **Automatic Posting by the OS**:
    * Threads performing I/O operations (e.g., `ReadFile`, `WriteFile`, or asynchronous socket operations) do **not directly post** `completion packets` to the IOCP
    * Instead, the operating system **automatically posts** a `completion packet` to the IOCP **when the I/O operation completes**
- **Manual Posting**
    * Any thread in the process can **explicitly post** a `task` to the IOCP queue using `PostQueuedCompletionStatus()`
    * This can be useful for **posting** custom tasks or signals **unrelated to I/O**
- **Worker Threads**
  - Separate threads (called `worker threads`) **block** on **`GetQueuedCompletionStatus()`** to **retrieve** `I/O completion packet` from the IOCP
  - These threads do **not** need to be the **same** threads that posted the `tasks` or initiated the `I/O operations`


## Supported I/O Functions

### [CreateIoCompletionPort()](https://learn.microsoft.com/en-us/windows/win32/fileio/createiocompletionport)
`CreateIoCompletionPort()` creates a `I/O completion port` with one or more `file handles`
```cpp
HANDLE WINAPI CreateIoCompletionPort(
  _In_     HANDLE    FileHandle,
  _In_opt_ HANDLE    ExistingCompletionPort,
  _In_     ULONG_PTR CompletionKey,
  _In_     DWORD     NumberOfConcurrentThreads
);
```
```cpp
#include <windows.h>
#include <iostream>

// Completion key structure to identify file handles
struct CompletionKey {
    HANDLE fileHandle;   // This can be removed
    std::string name;    // Identifier for the file handle
};

int main() {
    // Create an IO Completion Port
    HANDLE iocp = CreateIoCompletionPort(INVALID_HANDLE_VALUE, NULL, 0, 0);
    if (!iocp) {
        std::cerr << "Failed to create IOCP. Error: " << GetLastError() << "\n";
        return 1;
    }

    // Example file handle (substitute with a valid overlapped file or socket handle in real use)
    HANDLE fileHandle = CreateFile(
        "example.txt",
        GENERIC_READ,
        0,
        NULL,
        OPEN_EXISTING,
        FILE_FLAG_OVERLAPPED,       // This is required if you want to implement IOCP
        NULL
    );

    if (fileHandle == INVALID_HANDLE_VALUE) {
        std::cerr << "Failed to open file. Error: " << GetLastError() << "\n";
        return 1;
    }

    // Create a CompletionKey to associate with the file handle
    CompletionKey* key = new CompletionKey{fileHandle, "File1"};

    // Associate the file handle with the IO Completion Port
    if (!CreateIoCompletionPort(fileHandle, iocp, reinterpret_cast<ULONG_PTR>(key), 0)) {
        std::cerr << "Failed to associate handle with IOCP. Error: " << GetLastError() << "\n";
        CloseHandle(fileHandle);
        return 1;
    }

    // do sometihng here...

    // Clean up
    CloseHandle(fileHandle);
    CloseHandle(iocp);
    delete key;

    return 0;
}

```
- **Parameters**
    * `FileHandle`
        + An **open** `file handle` or `INVALID_HANDLE_VALUE`
        + If `INVALID_HANDLE_VALUE` is specified, the function creates an `I/O completion port` **without** associating it with a f`ile handle`
            - In this case, the `ExistingCompletionPort` parameter **must** be `NULL` and the `CompletionKey` parameter is **ignored**
    * `ExistingCompletionPort`
        + A `handle` to an **existing** `I/O completion port` or `NULL`
        + If this parameter specifies an **existing** I/O completion port, it does **not create a new** `I/O completion port`
            - the function associates it with the `handle` specified by the `FileHandle` parameter
            - it **returns** the `handle` of the **existing** `I/O completion port` if **successful** 
        + If this parameter is `NULL`, the function **creates a new** `I/O completion port`
            - the function also associates it with the `handle` specified by the `FileHandle` parameter
            - it **returns** the `handle` to the **new** `I/O completion port` if **successful**
    * `CompletionKey`
        + A **user-defined** value to **identify** the specific `file handle`
        + You can use **any user-defined** (custom) class or data structure for the `CompletionKey` as long as it can be **safely cast** to and from the `ULONG_PTR` type
        + This `completion key` should be **unique** for **each** `file handle`, and it accompanies the file handle throughout the internal completion queuing process. 
    * `NumberOfConcurrentThreads`
        + The `NumberOfConcurrentThreads` parameter specifies the `concurrency value` of a completion port
        + This value **limits** the `number of runnable threads` associated with the completion port
        + When the `total number of runnable threads` associated with the completion port **reaches** the `concurrency value`
            - the system **blocks** the execution of any subsequent threads associated with that completion port until the `number of runnable threads` **drops below** the `concurrency value`
        + The **best** overall maximum value to pick for the `concurrency value` is the `number of CPUs` on the computer
        + You can experiment with the concurrency value in conjunction with profiling tools to achieve the best effect for your application
- As mentioned above, when an asynchronous **I/O operation** on one of these `file handles` **completes**
    * an `I/O completion packet` is **queued** in first-in-first-out (FIFO) order to the associated I/O completion port
- An `I/O completion port` is associated with the process that created it and is **not sharable** between `processes`
    * However, it is **sharable** between `threads` in the `same process`

### `CloseHandle()`
```cpp
BOOL CloseHandle(
  [in] HANDLE hObject
);
```
```cpp
// Clean up
CloseHandle(fileHandle);
CloseHandle(iocp);
```
- **Parameter**
    * `hObject`
        + A `handle` to close
- The `I/O completion port handle`, and every `file handle` associated with it, serve as **references** to the `I/O completion port`  
- The `I/O completion port` is **freed** only when `all its references` are **removed**  
    * To **release** the `IOCP` and its associated system resources, `all related handles` must be properly **closed** 
    * You can do so by calling the `CloseHandle()` function

### [GetQueuedCompletionStatus()](https://learn.microsoft.com/en-us/windows/win32/api/ioapiset/nf-ioapiset-getqueuedcompletionstatus)
The `GetQueuedCompletionStatus()` function allows a thread to **wait** for a `completion packet` to be **queued** to the I/O completion port **instead of** directly **waiting** for an asynchronous `I/O operation` to **complete**
```cpp
BOOL GetQueuedCompletionStatus(
  [in]  HANDLE       CompletionPort,
        LPDWORD      lpNumberOfBytesTransferred,
  [out] PULONG_PTR   lpCompletionKey,
  [out] LPOVERLAPPED *lpOverlapped,
  [in]  DWORD        dwMilliseconds
);
```
```cpp
// some codes...

DWORD bytesTransferred;
ULONG_PTR completionKey;
LPOVERLAPPED overlapped;

if (GetQueuedCompletionStatus(iocp, &bytesTransferred, &completionKey, &overlapped, INFINITE)) {
    // Process the completion packet
    CompletionKey* retrievedKey = reinterpret_cast<CompletionKey*>(completionKey);
    std::cout << "Completed I/O for: " << retrievedKey->name << "\n";
} else
    std::cerr << "Failed to get queued completion status. Error: " << GetLastError() << "\n";
```
- **Parameters**
    * `CompletionPort`
        + A `handle` to the `completion port`
    * `lpNumberOfBytesTransferred`
        + A `pointer` to a variable that receives the `number of bytes` **transferred** in a **completed** `I/O operation`
    * `lpCompletionKey`
        + A `pointer` to a variable that receives the `completion key` value associated with the `file handle` whose I/O operation has completed
    * `lpOverlapped`
        + A `pointer` to a variable that receives the address of the `OVERLAPPED` structure that was specified when the completed I/O operation was started
    * `dwMilliseconds`
        + The `timeout` value to specify the number of `milliseconds` the caller is willing to **wait** for a completion packet to appear at the completion port
        + If a completion packet does **not appear** within the specified time, 
            - it returns `FALSE`, and sets `*lpOverlapped` to `NULL`
        + If it is `INFINITE`
            - the function will **never time out**
        + If it is `zero` and there is **no** `I/O operation` to dequeue
            - the function will **time out immediately**
        + Windows 8 and newer, Windows Server 2012 and newer
            - The `dwMilliseconds` value does **not include** time spent in `low-power states`
            - This means that the timeout does not continue counting down while the computer is asleep.
- While **any number** of `threads` can call `GetQueuedCompletionStatus()` for a **same** `I/O completion port`
    * The first time a thread calls `GetQueuedCompletionStatus()` with `that port`, it becomes **unable** to associated with **other** `I/O completion port` until one of the following happens
        + The thread **closes** the I/O completion port
        + The thread **exits**
    * In other words, a **single** `thread` can only be associated with **one** `I/O completion port` at a time
- When a thread in the `wait` state **resumes**, the `number of active threads` may **briefly exceed** the `concurrency value`
    * The system **quickly reduces** the `number of active threads` by preventing new ones from starting until it falls below the `concurrency value`
    * To handle this, it's recommended to **create more threads** in the `thread pool` than the `concurrency value`
        + A good rule of thumb is to have at least **twice** as many threads as the `number of processors` on the system

### `PostQueuedCompletionStatus()`
Threads can use the PostQueuedCompletionStatus function to place completion packets in an I/O completion port's queue
```cpp
BOOL WINAPI PostQueuedCompletionStatus(
  _In_     HANDLE       CompletionPort,
  _In_     DWORD        dwNumberOfBytesTransferred,
  _In_     ULONG_PTR    dwCompletionKey,
  _In_opt_ LPOVERLAPPED lpOverlapped
);
```
- **Parameters**
    * `CompletionPort`
        + A `handle` to an `I/O completion port` to which the I/O completion packet is to be posted
    * `dwNumberOfBytesTransferred`
        + The value to be **returned** through the `lpNumberOfBytesTransferred` parameter of the `GetQueuedCompletionStatus()` function
    * `dwCompletionKey`
        + The value to be **returned** through the `lpCompletionKey` parameter of the `GetQueuedCompletionStatus()` function
    * `lpOverlapped`
        + The value to be **returned** through the `lpOverlapped` parameter of the `GetQueuedCompletionStatus()` function
- With this function, the `completion port` can be used to **communicate** with `other threads` that do **not** perform `I/O operations`

### [WSASend()](https://learn.microsoft.com/en-us/windows/win32/api/winsock2/nf-winsock2-wsasend)
`WSASend()` is the **Windows-specific** version of the standard [send()](https://sadoe3.github.io/socket/socket-chapter5/#send) function used for sending data over a network socket
```cpp
int WSAAPI WSASend(
  [in]  SOCKET                             s,
  [in]  LPWSABUF                           lpBuffers,
  [in]  DWORD                              dwBufferCount,
  [out] LPDWORD                            lpNumberOfBytesSent,
  [in]  DWORD                              dwFlags,
  [in]  LPWSAOVERLAPPED                    lpOverlapped,
  [in]  LPWSAOVERLAPPED_COMPLETION_ROUTINE lpCompletionRoutine
);
```
- **Parameters**
    * `s`
        + A `descriptor` that identifies a connected `socket`
    * `lpBuffers`
        + A `pointer` to an array of `WSABUF` structures
        + Each `WSABUF` structure contains a `pointer` to a `buffer` and the `length`, in bytes, of the buffer
        + For a `Winsock` application, once the `WSASend()` function is **called**, the `system` **owns** `these buffers` and the `application` may **not access** them
        + This array must remain **valid** for the duration of the send operation
    * `dwBufferCount`
        + The `number of WSABUF structures` in the `lpBuffers` array
    * `lpNumberOfBytesSent`
        + A `pointer` to the number, in bytes, **sent** by this call if the I/O operation completes immediately
        + Use `NULL` for this parameter if the `lpOverlapped` parameter is **not** `NULL` to **avoid** potentially **erroneous results**
            - This parameter can be `NULL` **only if** the `lpOverlapped` parameter is **not** `NULL`
    * `dwFlags`
        + The `flags` used to **modify the behavior** of the `WSASend()` function call
        + For more information, check [this](https://learn.microsoft.com/en-us/windows/win32/api/winsock2/nf-winsock2-wsasend#using-dwflags) out
    * `lpOverlapped`
        + A `pointer` to a `WSAOVERLAPPED` structure
        + This parameter is **ignored** for **nonoverlapped** `sockets`
    * `lpCompletionRoutine`
        + A `pointer` to the `completion routine` **called** when the `send` operation has been **completed**
        * This parameter is **ignored** for **nonoverlapped** `sockets`
- **Return Value**
    * If **no error** occurs and the `send` operation has **completed immediately**
        + `WSASend()` returns `zero`
        + In this case, the `completion routine` will have already been **scheduled** to be called once the `calling thread` is in the `alertable` state
    * Otherwise, a value of `SOCKET_ERROR` is returned
        + and a `specific error code` can be retrieved by calling `WSAGetLastError()`
        + The error code `WSA_IO_PENDING` indicates that the overlapped operation has been **successfully initiated** and that **completion** will be indicated at a **later time**
        + `Any other error code` indicates that the overlapped operation was **not successfully** initiated and no completion indication will occur
- **Code Example**
    * You can view the example [here](https://learn.microsoft.com/en-us/windows/win32/api/winsock2/nf-winsock2-wsasend#example-code)



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}