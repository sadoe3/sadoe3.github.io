---
title: "Windows Programming : Thread Scheduler"

categories:
    - windows

tags:
    - [C++, System Programming, Windows, OS, Concurrency, thread, thread scheduler, priority, affinity]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-12-18
---

# Windows Programming : Chapter 1

> 이 포스트는 `Concurrent Programming on Windows (1st ed.)`을 바탕으로 작성되었습니다.

## Basic Understanding

### Thread-Based Scheduling
The **`Thread Scheduler`** in Windows is strictly **thread-based**, not process-based
- This distinction is crucial
- For instance, consider the case where you have `two processes` running
  - `One process` has **9 threads** running.
  - `The other process` has only **1 thread**
  - Both processes have **equal** `priority` (which will be explained soon)
- In this scenario, it's important to note that `each process` will **not** receive an **equal** amount of `processor time`
  - The `first process` will receive approximately **90%** of the `processor time`
  - The `second process` will receive the remaining **10%**
- This happens because the scheduler allocates time based on **threads**, not processes

### Thread States
A thread **transitions** through various `states` during its execution
- There are **7 primary** `states` for a thread
- **`Initialized` (`0`)**  
    * The thread is being allocated and **initialized** by the OS
- **`Ready` (`1`)**  
    * After initialization, the thread becomes ready to run (i.e., **runnable**)
    * and is added to the thread scheduler's `dispatcher database`
- **`Running` (`2`)**  
    * The thread is actively **running** on a processor
- **`Standby` (`3`)**  
    * The thread has been **selected** to run on a processor but has **not** yet **started execution**
        + At this point, the thread is **no longer** in the `dispatcher queue`
        + It may or may not transition to the `running` state
            - depending on whether it is **context-switched** out before it starts executing
- **`Terminated` (`4`)**  
    * The thread has **finished** executing its code
    * and will be **destroyed** once all outstanding `HANDLE` references are closed
- **`Waiting` (`5`)**  
    * The thread is **not** being considered for **execution** by the `thread scheduler`
        + This occurs when the thread
            - **voluntarily sleeps**
            - **waits** on a `kernel synchronization object` (e.g., `mutex`)
            - or performs **I/O operations**
    * Threads can also enter this state due to **suspension**
        + If a thread is created with the `CREATE_SUSPENDED` flag, its state is set directly to `waiting` from `initialized`
- **`Transition` (`6`)**  
    * The thread is **blocked** because the `required resource` is **unavailable**
    * It transitions back to `ready` when the `required resource` becomes **available**
- Although parsing a thread's state can be complex, you can monitor it via **performance counters**.
    * The `Thread\ThreadState` counter reports the **current state** of a specific thread
    * This can be accessed through `Performance Counter API`s or `Windows Performance Monitor` (`perfmon.exe`)

### Wait Reasons
Additionally, the `Thread\ThreadWaitReason` counter indicates the **reason** a thread is in the `waiting` state
- **`Executive` (`0`)**  
    * Waiting for a `kernel executive object` (e.g., `mutex`, `event`) to become **signaled**
- **`Free Page` (`1`)**  
    * Waiting for a **free** `virtual memory page`
- **`Page-In` (`2`)**  
    * Waiting for a `virtual memory page` to be **backed** by physical RAM (i.e., to be **paged into** memory)
- **`Page-Out` (`3`)**  
    * Waiting for a `virtual memory page` to be **paged out** to disk
- **`System Allocation` (`4`)**  
    * The OS is in the process of **allocating** a system resource required by the thread to continue execution
- **`Execution Delay` (`5`)**  
    * The thread's execution has been **delayed** by the OS
- **`Sleep` (`6`)**  
    * A request has been made to **explicitly** place the thread into a `waiting` state, usually via one of the `Thread Sleep API`s
- **`Event Pair High and Low` (`7`, `8`)**, **`LPC Receive and Send` (`9`, `10`)**  
    * These are used **internally** for **interprocess communication**


## Priority
The Windows thread scheduler is **priority-based**, meaning it prioritizes the execution of threads based on their assigned `priority`
- The scheduler will always **prefer** to run the thread with the **highest** `priority` in the system
    * It will **preempt** `lower-priority` threads that are already running when a `higher-priority` thread becomes **runnable**
- However, there's an **exception** to prevent **`starvation`**
  - `Starvation` occurs when a thread is **never executed** because `higher-priority` threads **continuously preempt** it
  - For example, a situation may arise where `higher-priority` threads are **always ready** to run, causing `lower-priority` threads to be **overlooked**
  - To resolve this, the operating system may allow a `lower-priority` thread to **run before** a `higher-priority` thread, ensuring fairness

### Two Components
From the scheduler's perspective, a thread's `priority` consists of two components
- **Process's Priority Class**  
    * This is the **default** `priority` assigned to the `process`
- **Thread's Relative Priority**  
    * This is an **offset** added to the process's default priority to determine the `thread`'s actual priority
- Together, these two components form a **numeric** `priority level` ranging from **1 to 31**
    * A **higher** `priority level` corresponds to a **higher** `priority`
- The priority levels from **1 to 15** are part of the **dynamic range**:
    * Threads in this range are typically used for **regular** applications, with dynamic priority adjustments based on system load
- The priority levels from **16 to 31** are part of the **real-time range**:
    * Threads in this range are used for **time-sensitive** tasks that require immediate processor access
        + but these threads can **starve** `lower-priority` threads due to their **high** `priority`

### I/O Prioritization
**I/O prioritization** in Windows determines the `priority` of **I/O operations**
- There are **5** `priority levels` for **I/O operations**
    * **Very Low**
    * **Low**
    * **Medium**
        + Default value
    * **High**
    * **Critical**
- You can **modify** the I/O priority of `processes` using the following methods
    * To **lower** the I/O priority of a process to **Very Low**
        + pass the `PROCESS_MODE_BACKGROUND_BEGIN` flag to `SetPriorityClass`
    * To **revert** the process I/O priority to its default value
        + pass `PROCESS_MODE_BACKGROUND_END` to `SetPriorityClass`
- For individual `threads`
    * To **lower** the I/O priority of a specific thread
        + pass the `THREAD_MODE_BACKGROUND_BEGIN` flag to `SetThreadPriority`
    * To **revert** the thread's I/O priority
        + pass `THREAD_MODE_BACKGROUND_END` to `SetThreadPriority`

### How to Access Priorities
**Priority class** of a `process` or **relative priority** of a `thread` can be managed in `Win32`
```cpp
BOOL WINAPI SetPriorityClass(HANDLE hProcess, DWORD dwPriorityClass);
DWORD WINAPI GetPriorityClass(HANDLE hProcess);

BOOL WINAPI SetThreadPriority(HANDLE hProcess, int nPriority);
DWORD WINAPI GetThreadPriority(HANDLE hProcess);
```
- They take a `HANDLE` to a target process as the `first parameter`
- Below table shows the `constants` used in `Win32` for **priority classes**

    |Title|`Win32` Constant Value|Level Range|Default|
    |:---:|:---|:---:|:---|
    |Real-time|`REAL_TIME_PRIORITY_CLASS`|16-31|24|
    |High|`HIGH_PRIORITY_CLASS`|11-15|13|
    |Above Normal|`ABOVE_NORMAL_PRIORITY_CLASS`|8-12|10|
    |Normal|`NORMAL_PRIORITY_CLASS`|6-10|8|
    |Below Normal|`BELOW_NORMAL_PRIORITY_CLASS`|4-8|6|
    |Idle|`IDLE`|1-6|4|
- Below table shows the `constants` used in `Win32` for **relataive priorities**

    |Title|`Win32` Constant Value|Level Modifier|
    |:---:|:---|:---:|
    |Time Critical|`THREAD_PRIORITY_TIME_CRITICAL`|**Absolute Value**: `31` for `real-time`, <br>`15` for `dynamic range`|
    |Highest|`THREAD_PRIORITY_HJGHEST`|+2|
    |Above Normal|`THREAD_PRIORITY_ABOVE_NORMAL`|+1|
    |Normal|`THREAD_PRIORITY_NORMAL`|0 (default)|
    |Below Normal|`THREAD_PRIORITY_BELOW_NORMAL`|-1|
    |Lowest|`THREAD_PRIORITY_LOWEST`|-2|
    |Idle|`THREAD_PRIORITY_IDLE`|**Absolute Value**: `15` for `real-time`, <br>`1` for `dynamic range`|
- It's strongly advised to **avoid** setting priorities above `normal` for both dynamic and real-time ranges
    * Doing so can **interfere with** `system services` that operate at `high` priorities
    * This may lead to **system instability** or **data corruption**
    * Therefore, most programs and threads should use the **default** priority level (`normal`/`normal`)


## Preemption
The Windows operating system uses **preemptive scheduling** (also known as **time-slicing**) for all threads, ensuring **fair** distribution of `processor time`
- The core idea is to give threads a **fair and roughly equal** amount of `execution time` based on the available hardware
    * When a thread runs, it is **preempted** if it exceeds its **`quantum`**
        + The `quantum` is a **specific period of time** allocated to a thread,
        + which can **vary** depending on the system (e.g., client vs. server OS) and can be configured
- If there are other threads **waiting** to execute when the `quantum` **expires**
    * The operating system may perform a **context switch** to allow another thread to run on the processor

### Quantums
In **Win32**, you can manage the `quantum` value **manually**, though it is typically controlled by the system
- **Client systems** typically have **shorter** `quantums` to enhance responsiveness and allow fairer scheduling of thread
    * However, **shorter** `quantums` can **degrade** overall performance
    * due to the overhead caused by more **frequent** `context switches`
- **Quantum accounting** occurs inside an `interrupt routine` within the operating system
    * When the `interrupt` **triggers**, the actively running thread's `quantum counter` is **decremented**
    * If the `quantum` **expires**
        + A `context switch` is **triggered**, and another thread may **preempt** the `current thread`
    * If the `quantum` has **not expired**
        + The thread **continues executing**
- When a thread **voluntarily blocks before** its `quantum` **expires**
    * It does **not** receive the **full** `quantum` after resuming
    * Instead, it resumes execution with the **remaining portion** of its original quantum

### Temporary Boost
In certain situations, a thread's `relative priority` can be **temporarily increased**
- Once **boosted**, the `priority` of the thread will **decrease** by `1` within the current `quantum` until it returns to its **original** `priority`
- The following circumstances might cause a **temporary boost** to a `thread's priority`

1. **Starvation**
    - If a service named `Windows Balance Set Manager` is active, a **temporary boost** might occur
        * This service runs **asynchronously** on a `system thread`
        * It monitors for **starved** threads
            + which are threads that have been in the `ready` state for **4 seconds or longer** without being executed
    - If the service **detects** a **starved** thread
        * It will assign the thread a **temporary boost**
    - It's worth noting that the `Windows Balance Set Manager` will always **boost** a starved thread's priority to level `15`
        * **regardless** of its **current** priority level

2. **Completion of Waiting**
    - When a thread **wakes up** because the `event` or `semaphore` it was waiting on has become **signaled**:
        * The thread receives a **temporary boost** of `+1`
    - It's worth noting that this **boost** is applied to the thread's `base priority`
        * If the thread is **already** enjoying a priority boost, the effect **will not** be cumulative

3. **GUI Thread**
    - When a `GUI thread` wakes up due to a new message being enqueued into its window's message queue
        * it gets a temporary boost of `+2`

4. **I/O Activity**
    - When a thread wakes up due to the completion of an **I/O Operation**
        * it receives a temporary priority boost of `+1`

5. **Foreground Process**
    - Whenever a thread in the `foreground process` **completes a wait** activity (defined by the process window currently in focus in **Explorer**)
        * It receives an additional priority boost of `+1` or `+2`, depending on system configuration
    - Unlike other priority boosts, this boost is **additive**
        * It is applied to the thread's **current** `priority`
            + **regardless** of whether it has **already** been boosted
    - Therefore, if the thread woke up due to an `event`, `semaphore`, `I/O`, or `GUI message`
        * It receives the `regular boost` **plus** this special `foreground priority boost`

6. **Quantum Boost**
    - On client OS SKUs
        * all threads in the `foreground process` receive **quantum boost** so long as the process remains in the foreground
    - This boost **multiplies** the `quantum` for all threads by `3`
    - So for example, instead of having a quantum of `2` clock ticks on client machines,
        * these threads have quantums of `6` clock ticks
    - This **reduces** `context switches` and allows the program to maintain responsiveness

- It's also **possible** to **manage** `priority boosting` in **dynamic range** in `Win32`
    ```cpp
    BOOL WINAPI SetThreadPriorityBoost(HANDLE hThread, BOOL DisablePriorityBoost);
    BOOL WINAPI GetThreadPriorityBoost(HANDLE hThread, PBOOL pDisablePriorityBoost);
    ```
    * If the `return value` is `true`
        + then it means **boosting** is **enabled**
    * This management **only applies**
        + `Event`
        + `Semaphore`
        + `I/O Activity`
        + and `GUI Thread Boost`
- Whereas, it is **not** possible to **turn off**
    + `quantum` **boosting**
    + **boosting** applied by the `Windows Balance Set Manager`
    + **boosting** thanks to `foreground process`


## Affinity
**CPU affinity** refers to the thread scheduler assigning a `specific thread` to **run** on a **particular subset** of a machine's `processors`
- Each thread has an `ideal processor`
    * If a `processor` is **free** and `multiple threads` are **available**
        + the scheduler will **prefer** the `thread` that has `that processor` as its `ideal processor` 
    * This improves **memory locality** and helps **evenly distribute** the workload across the machine
- If **no** `ideal processor` is available
    * the scheduler will choose the thread that was **most recently** run on a processor
    * however, this may incur **runtime costs**

### Setting Processor Affinity
You can **affinitize** `processes` or `threads` to a particular subset of processors, which can be **helpful** when used **correctly**
- However, if used **improperly**, performance can **degrade**
- `Process affinity` is inherited by `all threads` within `the process`
- Whereas, a `thread`'s **individual** `affinity` is **specific** to `that thread`
- `Process affinity` is also inherited by `other processes` created by the `original process`
    * For instance, if a `process` is affinitized to processors `0`, `1`, and `3`, `its threads` **cannot** be affinitized to processor `2`
    * they **can** only be assigned to any combination of processors `0`, `1`, and `3`

### Bit-Mask Representation of Affinity
- `Affinity` is **represented** as a `bit-mask`, where each bit corresponds to a processor
    * `0` means the processor is **not used**
    * `1` means the processor is available for **use**
- Suppose you are running on a `32-bit`, `8-CPU` machine
- To support `all processors`, the mask value would be: 
    * `0000 0000 0000 0000 0000 0000 1111 1111` (or `0xFF`)
- To use processors `0`, `2`, `3`, and `5`, the mask value would be
  `0000 0000 0000 0000 0000 0000 0010 1101` (or `0x25`)
- It's worth noting that bits should be read from **right to left**, where the least significant bit corresponds to processor `0`

### How to Assign Affinities
There are various ways to **assign** `CPU affinities`
- **`Win32` API**
    * The `Win32` API provides the functions `GetProcessAffinityMask()` and `SetProcessAffinityMask()` to programmatically retrieve and set the `affinity mask` for the given `process`
        ```cpp
        BOOL WINAPI GetProcessAffinityMask(HANDLE hProcess, PDWORD_PTR lpProcessAffinityMask, PDWORD_PTR lpSsystemAffinityMask);
        BOOL WINAPI SetProcessAffinityMask(HANDLE hProcess, DWORD_PTR dwProcessAffinityMask);
        ```
        ```cpp
        // simple use
        HANDLE hProcess = GetCurrentProces();
        SetProcessAffinity(hProcess, static_cast<DWORD_PTR>(0x25));     // use only processor 0, 2, 3, and 5

        DWORD_PTR pdwProcessMask, pdwSystemMask;
        GetProcessAffinity(hProcess, &pdwProcessMask, &pdwSystemMask);
        printf("process mask = %x, system mask = %x \n", pdwProcessMask, pdwSystemMask);
        ```
    * `Win32` also provides a way to set a specific `thread's affinity` by using `SetThreadAffinityMask()`  
        ```cpp
        DWORD_PTR WINAPI SetThreadAffinityMask(HANDLE hThread, DWORD_PTR dwProcessAffinityMask);
        // there's no GetThreadAffinityMask()
        ```
    * The `affinity mask` from a `thread` is only retrieved by the `return value` of the `SetThreadAffinityMask()` which was the **previous** mask value
- **Use Other Programs**
    * The other way is to use a tool that programmatically sets the `affinity`
    * There are 2 built-in tools to do this
        + `Windows Task Manager`
        + `START` command
    * Moreover, you can store a `process affinity mask` inside an executable's `PE` file image header
    * The book said `IMAGECFG.EXE` tool is helpful

### Ideal Processor
The `Win32` API provides a way to set the `ideal processor` for a specific thread
```cpp
DWORD WINAPI SetThreadIdealProcessor(HANDLE hThread, DWORD dwIdealProcessor);
```
- This function can be used in situations where
    * setting the processor `affinity` is **too restrictive**
    * but a higher-level component knows that running a `specific thread` **regularly** on a `particular processor` will **improve performance**
- The function returns the **previous** `ideal processor number`
    * You can retrieve the **current** `ideal processor`
        + by passing `MAXIMUM_PROCESSORS` as the `second argument`
    * If the function returns `-1`
        + it indicates a `failure`, typically due to **invalid arguments**


## Suspension
Windows provides the ability to **suspend** a thread's execution for an arbitrary length of time
- When a `thread` is **suspended**
    * The operating system regards it as `non-executable`, treating it as if the thread's `timeslice` has **expired** until it is **explicitly resumed**
    * The thread is **context-switched off** the processor and is no longer eligible to run
- When the `thread` is **resumed**
    * It is treated **similarly** to a thread that has **awakened** from an `I/O wait`
    * The thread is **placed back** into the `runnable queue` and will be **scheduled** to run on a processor

### `Win32` APIs for Suspending and Resuming Threads
- `CreateThread()` supports the `CREATE_SUSPENDED` flag, which ensures that the thread starts in the suspended state
    * The thread must be explicitly resumed before it can begin execution
- The `Win32` APIs to suspend and resume threads are `SuspendThread()` and `ResumeThread()`
    ```cpp
    DWORD WINAPI SuspendThread(HANDLE hThread);
    DWORD WINAPI ResumeThread(HANDLE hThread);
    ```
- Both functions take a `thread handle` as their `first argument` and
    * return a `DWORD`, which indicates the `suspension count` before the call
- Threads use a `counter` to manage multiple suspend calls on the same thread
    * When the it is **greater than** `0`, the thread **remains suspended**
    * Once the it **reaches** `0`, the thread is **resumed**
    * A `return value` of `-1` indicates an **error**
        + Detailed error information can be retrieved using `GetLastError()`

### Risks and Misuses of Thread Suspension
It's quite crucial to keep in mind that **suspension** should be used **cautiously**
- **Deadlock**
    * If the `thread` issuing the **suspension** doesn't know exactly what the target thread is doing, the target thread may be in the middle of `critical section`
    * For example, if `Thread A` **suspends** `Thread B` while `B` holds a `lock`, and then `A` attempts to acquire `that lock`, it will be blocked
        + `Thread A` could potentially **block indefinitely** unless it knows to resume `B` and wait for it to release `that lock`
- **Memory Leak**
    * If a thread is **suspended** and **never resumed**
        + it will remain in the suspended state
        + and `its resources` will **not** be **released** until the process exits
- **Misuse for Synchronization**:
    * Using thread **suspension** for **synchronization** is highly **discouraged**
    * This is because it can cause serious issues
        + Therefore use **waiting** or other functionalities appropriate for **synchronization**

### Appropriate Use Cases for Thread Suspension
The general advice is to **avoid** using suspension for **synchronization** and **only use** it in **controlled scenarios** where the thread's state is well-understood
- **Debugging and Profiling**
    * Thread suspension is **useful** in scenarios where
        + **information** about a `thread` needs to be captured **without executing** arbitrary program code while the thread is **suspended**
    * For instance, `Visual Studio` have a `freeze threads` feature that uses **thread suspension**, but only for short durations
        + This gathers information and then resume the thread quickly
- **Garbage Collection and Stack Tracing**:
    * The `Common Language Runtime` (`CLR`) uses **thread suspension** for stack walks to find `live references` during **garbage collection**
    * In some cases, programmatically capturing a `stack trace` requires temporarily suspending a thread
- **Short Duration Suspensions**:
    - In `debuggers` or `profiler` tools, **thread suspension** typically lasts
        * only for a brief moment
            + just long enough to gather the necessary data
        * and the thread is resumed immediately afterward


## Extra Details

### Multimedia Scheduler
Technically speaking, the `Multimedia Class Scheduler Service` (MMCSS) is **not** a `thread scheduler`
- It is a `service` running under `svchost.exe` at a `very high` priority that
    * **monitors** the activity of `multimedia programs*` registered with the system
- This service works in cooperation with these programs to **boost** their `priorities`, ensuring **smoother** multimedia playback
- Specifically, this service
    * **boosts** threads within multimedia programs into the **real-time range**.
    * **throttles** (reduces the priority) periodically to **avoid starving** other processes on the system
- To **register** your program with `MMCSS`
    * Add an entry to the following registry key:
        + `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\Tasks`
- For further details, please refer to **MSDN**.

### Sleeping and Yielding
Occasionally, a program may need to **temporarily remove** the `current thread` from the `Windows thread scheduler`'s **control**
- In `Win32`, there are three APIs that can be used to achieve this
    ```cpp
    VOID WINAPI Sleep(DWORD dwMilliseconds);
    DWORD WINAPI SleepEx(DWORD dwMilliseconds, BOOL bAlertable);
    BOOL WINAPI SwitchToThread();
    ```
- **`Sleep()`**
    * **Suspends** the current thread for the **specified duration**
    * If the `duration argument` is `0`, Windows will **remove** the `thread` from the processor
        + **only if** there is `another thread` **ready** to run with an **equal** or **higher** `priority`
        + This means that if `runnable threads` are at a **lower** `priority`
            - the `calling thread` will **continue running** instead of yielding to other threads
    * If the `duration` is **greater** than `0`, the `calling thread` will be **removed** from the scheduler's runnable queue for the **specified duration**
        + It's worth noting that the **resolution** of the `system clock` determines how **accurate** the sleep duration will be
        + For example, if the system clock's resolution is `10 milliseconds`, specifying a duration **less than** `10 milliseconds` will effectively **round up** to `10 milliseconds`
    * After the sleep period, the thread will be placed back in the runnable queue
- **`SleepEx()`**
    * **Suspends** the current thread like `Sleep()`
        + but allows for the possibility of **asynchronous procedure calls** (`APC`s)
    * Passing `TRUE` for the `bAlertable` allows `APC`s to dispatch, if any `APC`s are in the thread's `dispatch queue`
- **`SwitchToThread()`**
    * **Yields** the current processor to another thread for a short time
    * It **yields** the processor for a `single timeslice` to another thread if one is ready to run, **regardless** of the thread's `priority`
        + If **no** `other threads` are **ready** to run, the `calling thread` **remains** on the processor
- You can **adjust** the `system's timer granularity` using the `timeBeginPeriod` and `timeEndPeriod` APIs
  - However, modifying the timer granularity can **negatively affect** system performance and power consumption


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}