---
title: "IOCP : How to Handle Blocking"

categories:
    - iocp

tags:
    - [C++, System Programming, Windows, OS, IOCP, I/O Functions, Multi-Threading]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2025-02-08
---

# Windows Programming : Chapter 3

## How to Handle Blocking on GetQueuedCompletionStatus()
If you want to handle **termination** of the `worker thread`, you will need to introduce a mechanism to **interrupt** the `blocking call`, such as using an `event` or `signaling` from the main thread

### Steps to Fix
You can achieve this by using an **interrupt mechanism** like a `manual-reset event` or a `custom flag` that signals the worker thread to exit

1. Create a `manual-reset event`
    - This will be used to signal the worker thread to stop waiting for I/O completion
2. Modify the worker thread
    - Add a check for the event that will allow it to exit the `GetQueuedCompletionStatus()` call
3. Signal the event when the application is terminating

### Code Example
```cpp
#include <windows.h>
#include <iostream>
#include <thread>
#include <atomic>

// Flag to control thread termination
std::atomic<bool> keepRunning(true);  

// This will hold the completion port and event
HANDLE hCompletionPort;
HANDLE hExitEvent;

void workerThread() {
    OVERLAPPED* pOverlapped = nullptr;
    ULONG bytesTransferred = 0;
    ULONG completionKey = 0;
    
    while (keepRunning) {
        // Note that INFINITE is given
        BOOL result = GetQueuedCompletionStatus(hCompletionPort, &bytesTransferred, &completionKey, &pOverlapped, INFINITE);

        if (result) {
            // Process the completed I/O (this is just a placeholder)
            std::cout << "IO completed, bytes transferred: " << bytesTransferred << std::endl;
        } else {
            // Check if it was canceled due to the termination signal
            if (GetLastError() == ERROR_ABANDONED_WAIT_0 || !keepRunning) {
                std::cout << "Worker thread exiting..." << std::endl;
                break;
            }
        }
    }
}

void stopWorkerThread() {
    // Signal the event to stop the worker thread
    keepRunning = false;

    // If worker thread is waiting on GetQueuedCompletionStatus, interrupt it by signaling the exit event
    SetEvent(hExitEvent);
}

int main() {
    hCompletionPort = CreateIoCompletionPort(INVALID_HANDLE_VALUE, nullptr, 0, 1);

    // Create an event to signal the worker thread
    hExitEvent = CreateEvent(nullptr, TRUE, FALSE, nullptr);
    
    if (hCompletionPort == nullptr || hExitEvent == nullptr) {
        std::cerr << "Failed to create IOCP or event" << std::endl;
        return 1;
    }

    std::thread worker(workerThread);
    std::this_thread::sleep_for(std::chrono::seconds(5));

    // Stop the worker thread gracefully
    stopWorkerThread();

    worker.join();
    CloseHandle(hCompletionPort);
    // Note that Event should be closed as well
    CloseHandle(hExitEvent);

    return 0;
}
```
- `CreateEvent()` takes four parameters
    * `lpEventAttributes` (LPSECURITY_ATTRIBUTES)
        + This is a pointer to a `SECURITY_ATTRIBUTES` structure, which specifies **security attributes** for the event
        + If you do **not need** to specify custom security attributes, you can pass `nullptr`
            * When nullptr is passed, the event object is created with the **default** security attributes
        + Purpose
            * This parameter defines the security descriptor for the event, which determines who can access or modify it
            * If `nullptr` is passed, the event will be created with default security, and access is controlled by the **current security settings** of the process
    * `bManualReset` (BOOL)
        + This parameter specifies whether the event is a **manual-reset** event or an **auto-reset** event
        + `TRUE`
            * The event is a manual-reset event
            * Once the event is signaled (set to signaled state), it remains signaled until the application explicitly resets it
            * **Multiple** threads can wait for the event without it being automatically reset
        + `FALSE`
            * The event is an auto-reset event
            * After one thread is released from waiting, the event automatically returns to the non-signaled state
            * **Only one** thread can be released per signal
        + Purpose
            * Determines how the event behaves after being set to the signaled state
            * If TRUE, the event stays signaled until manually reset, and if FALSE, it automatically resets after a single thread is released
    * `bInitialState` (BOOL)
        + This parameter specifies the **initial state** of the event when it is created
        + `TRUE`
            * The event is created in the **signaled** state (ready for threads to proceed)
        * `FALSE`
            * The event is created in the **non-signaled** state (threads that wait for the event will block until it is signaled)
        * Purpose
            * Defines whether the event starts in a signaled state or not
    * `lpName` (LPCTSTR)
        + This is the **name** of the event object
        + If the event is meant to be used only within the current process, this parameter can be `nullptr` to create an unnamed event
        + If you want the event to be named, specify a **unique** name
        + Purpose
            * Allows you to create a named event that can be opened by other processes
            * Named events are useful for inter-process synchronization

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}