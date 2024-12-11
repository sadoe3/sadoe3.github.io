---
title: "C++ Concurrency : Testing and Debugging"

categories:
    - cpp

tags:
    - [C++, Programming Language, Concurrency, test, debug]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-12-11
---

# C++ Concurrency : Chapter 11

> 이 포스트는 `C++ Concurrency in Action (2nd ed.)`을 바탕으로 작성되었습니다.

## Concurrency-Related Bugs
In order to test or debug the concurrency issue, you must understand the problem itself first

### Unwanted Blocking
**Unwanted blocking** typically occurs when one thread is **waiting** for the thread which was **already blocked** to perform an action, causing the second thread to be blocked as well
- There are three cases of this issue
- **Deadlock**  
    * A situation where one thread is waiting for another thread, and the second thread is waiting for the first, resulting in a circular dependency
- **Livelock**  
    * It's similar to **deadlock**, but a key difference is that the threads are **not** in a **blocking** state
    * Instead, they are actively checking in a loop, such as in a **spin lock**, without making progress
- **Blocking due to I/O or external input**  
    * If a thread is blocked while waiting for external input, it cannot proceed, even if the expected input never arrives

### Race Condition
**Race conditions** are a common source of issues in multithreaded programs
- They typically lead to the following types of problems
- **Data Race**  
    * A specific type of race condition that results in **undefined behavior** due to **unsynchronized** concurrent access to a **same** `shared memory location`
- **Broken Invariants**  
    * This includes scenarios such as
        + **Dangling pointers**
            - Occurs when another thread deletes data that is still being accessed
        + **Random memory corruption**
            - Happens when a thread reads inconsistent values due to partial updates
        + **Double-free errors**
            - For instance, when two threads simultaneously pop the same value from a `queue` and both attempt to delete the associated data
    * These cases typically lead to **undefined behavior**
- **Lifetime Issues**  
    * This arises when a thread **outlives** the data it accesses
        + resulting in the thread accessing data that has already been deleted or destroyed
    + In some cases, the memory may even be reused for another object

## How to Test
Now, let's locate these bugs so that we can fix them

### Code Review
The most straightforward way to locate issues is by **reviewing** the code itself
- However, When you review your own code, it's easy to see your intended logic rather than what is actually there
    * To resolve this, it is highly recommended to have **someone else** review your code
    * If that's not possible, take an **adequate break** before reviewing it yourself to give yourself some distance and **reduce familiarity** with your own code
- When reviewing the code on your own, try to **explain** how it works **in detail**
    * Ask **yourself** questions about the code and make sure to explain the answers to ensure clarity and thorough understanding
    * As you explain, think about each line: What could happen? Which data does it access? And so on.
- Try these questions when reviewing multithreaded code
    * Which data needs to be protected from concurrent access?
    * How do you ensure that the data is protected?
    * Where in the code could other threads be at this time?
    * Which mutexes does this thread hold?
    * Which mutexes might other threads hold?
    * Are there any ordering requirements between the operations done in this thread and those done in another? How are those requirements enforced?
    * Is the data loaded by this thread still valid? Could it have been modified by other threads?
    * If you assume that another thread could be modifying the data, what would that mean and how could you ensure that this never happens?

### Things to Consider When Testing
After reviewing your code, it's essential to **verify** that there are no bugs present
- However, testing `multithreaded` code is more **challenging** than testing `single-threaded` code
    * This is because the `results` can **vary** from run to run due to the indeterminate scheduling of threads
        + This means the program might work correctly in one run, but fail in another
- Therefore, it's recommended to **eliminate concurrency** from the test to **verify** whether the issue is **related to concurrency**. 
    * This is because a bug in the multithreaded portion of your application does not necessarily imply that it is concurrency-related
- If the problem is indeed related to concurrency, it's best to test the **smallest** portion of code that could potentially reveal the issue
    * For example, test the concurrent `queue` **directly** instead of testing it through an **entire system** that uses the `queue`
- When testing, aim to cover as many possible scenarios as you can, ideally testing **all possible cases**
    * If you test a ``queue``, then you need to think the cases like
    * One thread calling `push()` or `pop()` on its own to verify that the `queue` works ata basic level
    * One thread calling `push()` on an empty `queue` while another thread calls `pop()`
    * Multiple threads calling `push()` on an empty `queue`
    * Multiple threads calling `push()` on a full `queue`
    * Multiple threads calling `pop()` on an empty `queue`
    * Multiple threads calling `pop()` on a full `queue`
    * Multiple threads calling `pop()` on a partially full `queue` with insufficient items for all threads
    * Multiple threads calling `push()` while one thread calls `pop()` on an empty `queue`
    * Multiple threads calling `push()` while one thread calls `pop()` on a full `queue`
    * Multiple threads calling `push()` while multiple threads call `pop()` on an empty `queue`
    * Multiple threads calling `push()` while multiple threads call `pop()` on a full `queue`
- Next, you should take into account other factors related to the **test environment**
    * What you mean by `multiple threads` in each case
        + `3`, `5`, `60`?
    * Whether there are enough processing cores in the system for each thread to run on its own core
    * Which processor architectures the tests should be run on
    * How you ensure suitable scheduling for the **while parts** of your tests
- It's important to note that library calls may use internal variables to store state, which becomes shared if multiple threads use the same set of library calls 
    * This can be problematic because it's not always obvious that the code is accessing shared data
    * To address this, you can either add proper protection and synchronization or use alternative functions that are safe for concurrent access from multiple threads

### Design Testable Code
It's essential to **think** about how to **test** the code **before** you write it
- In general, code is easier to test when the following factors are in place
    * The **responsibilities** of each function and class are **clear**
    * The functions are **concise** and **focused**
    * Your tests can have **full control** over the environment in which the code is executed
    * The **code** responsible for the **specific operation** being tested is located **close together**, rather than being scattered throughout the system

### Multithreaded Testing Techniques
There are **three key techniques** for testing multithreaded code
- **Stress Testing**  
    * The main objective of **stress testing** (or brute-force testing) is to **push** the code to its **limits** to see if it breaks
    * This involves running the code multiple times, possibly with many threads executing simultaneously
        + The more you run the test, the greater your **confidence** in the code's stability
    * The **downside** is that it can give you **false confidence**
        + There may be cases where the test does **not trigger** a bug in `your system environment`, but it **could occur** `in others`
        + This is why it's important to consider the **`system architecture`** when testing
    * While stress testing can build confidence, it does **not guarantee** that all issues will be found
- **Combination Simulation Testing**  
    * This technique uses `software` to **simulate** the runtime environment of your code, recording data accesses, locks, and atomic operations from each thread
    * It then runs the code with all possible combinations of operations, using the C++ memory model to identify race conditions and deadlocks
    * While this approach **guarantees** detection of system-specific issues
        + it can be **time-consuming** for non-trivial programs due to the **exponential growth** of combinations with more threads and operations
        * Therefore, it's **best** for testing **small, specific** code segments
    * Additionally, it **requires** simulation `software` that supports the operations in your code
        + If you don't have it, you can't use this approach
- **Testing With A Special Library**  
    * This approach involves using a library implementation of synchronization primitives, such as `std::mutex` or `std::condition_variable`, to identify issues
    * For example, by checking which mutexes were locked when data was accessed
        + you can verify whether the appropriate mutex was correctly locked by the calling thread during data access

### Structure of Test Code
- When writing `test code`, it's important to identify the **four distinct parts** of each test
    * **General setup code**
        + Code that runs first, where you can **construct** and **initialize** the data to be tested
    * **Thread-specific setup code**
        + Code that msut run on each thread, allowing you to **synchronize** multiple threads for testing
    * **Concurrent code**
        + Code that runs on each thread concurrently, where you perform the **actual testing**
    * **Post-execution code**
        + Code that runs after all threads finish, where you can use `assert()` to verify the state of the system
- The following code demonstrates this structure
    ```cpp
    // test for concurrent `push()` and `pop()`
    void testConcurrentPushPopOnEmpty`queue`() {
        // step 1
        `queue`ThreadSafe<int> my`queue`;
        std::promise<void> go, readyPush, readyPop;
        std::shared_future<void> ready(go.get_future());
        std::future<void> donePush;
        std::future<int> donePop;

        try {
            donePush = std::async(std::launch::async, [&my`queue`, ready, &readyPush]() {
                    // step 2
                    readyPush.set_value();
                    ready.wait();
                    // step 3 - test it!!
                    my`queue`.push(42);
                });
            donePop = std::async(std::launch::async, [&my`queue`, ready, &readyPop]() -> int {
                    // step 2
                    readyPop.set_value();
                    ready.wait();
                    // step 3 - test it!!
                    return my`queue`.`pop()`;
                });

            // step 2
            readyPush.get_future().wait();
            readyPop.get_future().wait();
            // step 3 - test it!!
            go.set_value();

            // step 4
            donePush.get();
            assert(donePop.get() == 42);
            assert(my`queue`.empty());
        }
        catch (...) {
            // excpetion handling
            go.set_value();
            throw;
        }
    }
    ```

### Test the Performance
Since one of the main reasons for using concurrency is to improve performance, it's essential to **test** whether the **performance** actually improves
- The key focus should be on verifying **`scalability`**  
    * Test the code starting with a single thread and gradually increase the number of threads to see if performance (e.g., calculation time) improves as more threads are added
    * If performance improves, it indicates that your program benefits from concurrency


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}