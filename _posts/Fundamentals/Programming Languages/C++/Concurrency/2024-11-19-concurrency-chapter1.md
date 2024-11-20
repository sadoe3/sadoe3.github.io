---
title: "C++ Concurrency : title"

categories:
    - cpp

tags:
    - [C++, Programming Language, Concurrency]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-19
---

# C++ Concurrency : Chapter 1

> 이 포스트는 `C++ Concurrency in Action (2nd ed.)`을 바탕으로 작성되었습니다.


## History of C++ Standard Thread Library

### Pre-C++11: Custom Libraries for Multithreading
Before the introduction of the C++11 standard, developers relied on custom libraries to implement multithreaded programs.
- These libraries were often platform-specific and required manual handling of low-level threading mechanisms, leading to portability and maintainability challenges.

### C++11: Standardization of Multithreading
The `C++11` standard marked the **first** official recognition of multithreaded programming within the C++ language specification.
- This was a pivotal moment, allowing developers to write multithreaded applications using a standardized approach without the need for third-party libraries.
- The goal was to provide a **portable and reliable** mechanism for `multithreading` across different platforms, significantly improving the ease of writing concurrent programs.
- The standard thread library in C++11 was heavily influenced by prior experience accumulated through the use of custom libraries.
    * Notably, **Boost** served as the primary model for the C++11 threading library, integrating proven techniques and patterns to ensure a robust and efficient implementation.

### C++14/C++17: Enhancing Performance and Flexibility
The subsequent revisions of the C++ standard, namely `C++14` and `C++17`, continued to refine and expand upon the threading facilities.
- These updates introduced **additional functionality** designed to enhance the **performance** of multithreaded applications while **freeing** developers from the complexities of **platform-specific** assembly code.
- The new features provided developers with better control over thread synchronization, management, and scheduling, while also allowing for more efficient handling of concurrent tasks.
- C++14 and C++17 continued the trend of enabling developers to focus on writing portable, high-performance code without worrying about the intricacies of low-level, platform-specific threading implementations.


## What is Concurrency?

### Basic Definition
Concurrency refers to the concept of **two or more separate activities occurring simultaneously**.
- In the context of **computers**, this means that a single system is capable of performing multiple independent activities in parallel.

### Task Switching: Emulating Concurrency
In many systems, particularly those with a **single processing unit** (such as a single-core CPU), **`task switching`** is employed.
- A single processor performs one task at a time, but it can switch between tasks many times per second.
- This rapid switching creates the **`illusion` of concurrency**, where it appears that multiple tasks are being executed simultaneously
    * even though only one task is being processed at any given moment.

### Genuine Hardware Concurrency
**`Genuine` hardware concurrency** occurs when a system has **multiple processing units** (such as multiple cores or threads), each capable of executing independent tasks simultaneously.
- In such systems, tasks can be **truly** executed in parallel, without the need for switching, leading to **true concurrency**.

### Task Switching in Modern Systems
Although modern desktop systems often contain multiple threads and processing cores, the `number of tasks` that need to be executed concurrently is frequently much **larger** than the available `number of threads`.
- As a result, **`task switching`** is still a **vital mechanism**, where tasks are distributed across available threads


## Concurrency with Multiple `Processes` (Single Thread)

### Pros:
**`True` Independent Execution**
- Each process runs independently, which ensures that tasks are isolated from each other.
- This allows processes to operate independently, potentially on separate systems across multiple cores in a `single machine` or even `multiple machines` within a network.

### Cons:
**Complex Communication**
- Communication between processes (typically through Inter-Process Communication, or `IPC`) can be both **complicated** and **slow**.
- This is because processes operate in **separate memory spaces**, requiring more **overhead** for data exchange.


## Concurrency with Multiple `Threads` (Single Core)

### Overview:
A **thread** is often described as a **lightweight process**.
- While threads run **independently**, they **share** the **same** `address space`

### Pros:
**Efficient Communication**
Since threads **share** the **same** `memory space`, communication between them is typically **easier** and **faster** compared to processes.
- This reduces the overhead associated with data sharing and synchronization.
  
### Cons:
**Data Consistency Challenges**
- A major concern when using threads is ensuring that `shared data` remains **consistent** when accessed by multiple threads concurrently.
- **Without** proper synchronization, this can lead to `race conditions` and `unpredictable behavior`.

### Mainstream Usage:
Due to the **lack** of **intrinsic support** for true concurrency across `multiple cores` in the **C++ standard**
- **`Multithreaded` concurrency** within a single core is the more common approach.
- To **implement** true concurrency across `multiple cores`
    * developers often rely on **third-party libraries** (such as `OpenMP`, `Intel TBB`, or `Boost`) to take full advantage of hardware capabilities.


## Parallelism vs. Concurrency
While the terms `parallelism` and `concurrency` are often used **interchangeably**, they refer to **distinct** concepts in computing, each with its own focus and implications.

### Brief Explanation
**`Parallelism`** is about **performance optimization**:
- It aims to make tasks run **simultaneously** on multiple cores or processors to speed up execution and reduce overall processing time.

**`Concurrency`** is concerned with the **separation and management** of tasks:
- It focuses on **coordinating and structuring tasks** so that independent or unrelated parts of a program can make progress without unnecessary blocking, even if they are not executed at the same time.










## Reasons for Concurrency
Concurrency is driven by **two** main goals: **separation of concerns** and **performance**.

### Separation of Concerns
Separation is almost always beneficial in programming.
- By **grouping related code together** and keeping **unrelated code apart**, you can:
    * Make your programs **easier to understand and maintain**.
    * **Simplify** `testing`.
    * **Reduce** the likelihood of `bugs`.
- **Without** concurrency, running multiple tasks simultaneously would require either:
    * Writing a `task-switching` framework **yourself**.
    * Actively making calls to **unrelated** areas of code
- Moreover, you can also **implement** things that would be difficult or **impossible** in a single-threaded environment by using concurrency.
    * A **typical example** is an **active loading screen** that displays while the application continues to load data in the background on a different thread.

### Performance
Concurrency can enhance performance in **two primary ways**:

#### **Task Parallelism**
The first approach, which is known as **task parallelism**, handles a `single task` with `multiple threads`, reducing the total runtime of that single task.
- It can vary in what it separates:
    * Different threads perform **different** parts of the **algorithm** on **same data**.
    * Each thread performs the **same operation** on **different** parts of the **data**.

#### **Data Parallelism**
The second approach, which is known as **data parallelism**, uses concurrency to handle `multiple tasks` with `multiple threads` at once, such as processing `multiple files` in parallel.
- It enables the processing of large problems more efficiently
    * e.g., applying a `same function` to `each file` in a large dataset


## When Not to Use Concurrency
If you need to achieve something that **can't be done** in a `single-threaded` environment, such as implementing an **active loading screen**, use it for sure
- otherwise, the **basic principle** is simple
    * *the `benefit` of using concurrency must **outweigh** its `costs`*.

### When to Avoid Concurrency
If the `performance` improvements, `readability`, or `maintainability` of the code **aren't significantly enhanced**, it's better to **avoid** introducing `concurrency`.
- Introducing `concurrency` with a **lack of benefits** can often **complicate** the code, making it **harder** to **understand** and more prone to **bugs**.
- It's note worting that the `performance` gain might **not** be as significant as **anticipated**
    * since the **operating system** needs to handle `additional tasks`
        + such as `scheduling threads` and `managing context switches`
    * to implement threading.

### Risks of Overusing Concurrency
Using **too many** `threads` can quickly **exhaust** available **memory** and **slow** down the **entire system**. 
- This is because the **operating system** needs to perform **more context switches** between threads
    * this adds overhead and potentially makes the system **less efficient**.
- Therefoer, in order to achieve the **best** possible `performance`
    * it is important to **limit** `the number of threads` running at once.


## Extra Details

### Embarrassingly Parallel Algorithm
An **embarrassingly parallel algorithm** is one that can be **easily divided** into `independent threads`.
- As the `number of available threads` **increases**, the `performance` of these algorithms can **scale effectively**, leading to significant performance improvements.
- However, for **other types of algorithms**, you may need to set a `fixed number` of parallel tasks, meaning they are **less scalable**.

### Platform-Dependent Operations
While the **standard library** provides reasonably **comprehensive facilities** for `multithreading` and `concurrency`, there are situations where you may need to use `platform-specific` **operations**. 
- In such cases, the library offers the `native_handle()` member function
    * which allows you to access the underlying **platform-specific thread handle**.
- This enables you to interact with `low-level, platform-specific` **operations** that might not be covered by the standard library.
- By using `native_handle()`, you can utilize `platform-specific` features or interact with `external APIs` that require direct access to the operating system’s threading model.
    * However, keep in mind that using platform-specific functionality can **reduce** the `portability` of your code.


## Summary
In `C++`, using `multiple threads` **isn't complicated** in and of **itself**
- The **complexity** lies in `designing` the code so that it `behaves as intended`

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}