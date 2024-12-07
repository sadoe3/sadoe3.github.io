---
title: "C++ Concurrency : Performance"

categories:
    - cpp

tags:
    - [C++, Programming Language, Concurrency, concurrent code, performance, dividing work, data contention, false sharing]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-12-05
---

# C++ Concurrency : Chapter 8

> 이 포스트는 `C++ Concurrency in Action (2nd ed.)`을 바탕으로 작성되었습니다.

## Work Division
To write efficient **multi-threaded** code, it is crucial to **divide** the workload across multiple threads
- The method you choose for dividing the work will **significantly impact** both the `performance` and the `clarity` of your code
- There are three primary approaches to dividing the work

### Dividing Work by N Elements per Thread
When performing operations such as `std::for_each()` on each element
- A straightforward method for workload distribution is to allocate `a fixed number of elements` (N) to `each thread`
- The first thread processes the first N elements, the second thread processes the next N elements, and so on

### Recursive Division
A more sophisticated approach involves **recursively** dividing the work
- This method is commonly used in `divide-and-conquer` algorithms, such as [**quicksort**](https://sadoe3.github.io/cpp/concurrency-chapter4/#parallel-quicksort), which we have already implemented before
- By **recursively** breaking down the problem into smaller subproblems, the work can be more efficiently distributed across multiple threads

### Dividing Work Based on Task Type
Another approach to work division is based on the nature of the tasks themselves
- Suppose that there's a **pipeline** process in which the work is divided into a series of sequential steps
    + In this case, `each thread` performs **a specific** task in **a series of stages**
- Consider a video processing application that buffers `small segments` of video data, **rather than** loading the `entire` video at once
    * In this scenario, each thread is responsible for handling a specific portion of the video stream
        + For example:
        + `Thread A` processes `Chunk A`
        + `Thread B` streams the processed `Chunk A` to the user
        + While `Thread B` is streaming, `Thread A` begins processing `Chunk B`
        + This cycle continues, ensuring that the user can watch the video without delay, avoiding the need to wait for the entire video to load


## Factors Affecting Performance
Understanding the various factors that influence performance is crucial when optimizing multi-threaded applications

### Number of Threads in Use
- Using **too few** threads may lead to `underutilization` of available processing resources, resulting in wasted potential
- If **too many** threads are used, it can cause **frequent** `context switching` (also known as `oversubscription`), which can degrade performance due to the overhead of managing thread states
- Therefore, it is important to select the **appropriate number** of threads to balance efficiency and performance
    * Consideration should also be given to cases where multiple instances of the same program are running concurrently

### Data Contention
When `multiple threads` **access** the `same data` simultaneously, either to read or modify it, `data contention` occurs
- When threads `read` from the same data, **caching** mechanisms may be employed to ensure each thread has the `most recent copy`
- When threads `write` to the same data, synchronization mechanisms are required to ensure that the most recent value is properly `propagated`, allowing other threads to read it correctly
    * This contention can still occur even when using operations like `fetch_add()` with `relaxed` memory ordering 
    * The `fetch_add()` operation is a `read-modify-write` operation
        + meaning that when multiple threads invoke this operation, they may need to wait for one another, creating a scenario where threads are blocked
- `High contention` occurs when threads **frequently** wait for others
    * while `low contention` refers to situations where this is **rare**
- `High contention` can lead to **cache ping-pong**, where data are constantly passed between **CPU caches**, decreasing performance
    * This issue also applies to mutexes, where multiple threads compete for access to the same memory location

### False Sharing
Processors manage data in blocks known as **`cache lines`** (typically 32 or 64 bytes in size) rather than individual memory locations
- It's worth noting that **small** `data elements` located next to each other in memory may end up in the **same** `cache line`, even if they are **not shared** in your code
- This can lead to `false sharing`
    * where `multiple threads` unnecessarily compete for the **same** `cache line`
    * resulting in **cache ping-pong** and performance degradation
- In order to address this, structure your data so that
    * Data used by the **same** `thread` resides in the **same** `memory region`
        + and ideally in the **same** `cache line`
    * Data used by **different** `threads` is placed **far apart** in memory
    * You can understand how this works by reading the next sub chapter
        + it's related to `padding`
- You can use `std::hardware_destructive_interference_size`, which represents the maximum number of consecutive bytes that could be affected by `false sharing`
    * If your data structure **exceeds** this size, it is more likely to be **immune** to `false sharing`
- In some cases, having multiple data elements in the **same** `cache line` can actually **improve performance** — especially when **all the data** is accessed by the **same** thread
    * This is known as **`constructive interference`**
    * The size of the cache line can be managed using `std::constructive_interference_size`, which indicates the maximum number of consecutive bytes guaranteed to be placed within the **same** `cache line`
- By carefully designing your data layout, you can minimize false sharing and take advantage of cache locality to optimize multi-threaded performance.

### Data Access Patterns
The way data is accessed can significantly affect performance in multi-threaded applications
- Modifying the layout of the data or adjusting how data elements are distributed across threads can have a considerable impact on performance
- Efficient data access patterns can minimize contention and improve cache utilization, while poor patterns may lead to excessive synchronization or cache misses

### Scalability
A program is considered **`scalable`** when its **performance improves** as **more processing cores** are added to the system
- Scalable code efficiently leverages additional cores to process larger workloads without suffering from diminishing returns or bottlenecks
    * allowing the application to take full advantage of the hardware


## how to improve performance
There are some guidelines to follow in order to improve the performance

### Data Proximity
- There are 3 guidelines to follow regarding `memory location`
    * Adjust the data distribution between threads so that data **close together** is processed by the **same thread**
        + in order to **prevent** data processed by a single thread from being **scattered**
    * Ensure that data accessed by **separate threads** is sufficiently **far apart**
        + in order to avoid `false sharing`, using `std::hardware_destructive_interference_size` as a hint
    * Minimize the data required by any given thread
- When a class contains both a mutex and some **small** data, they may reside in the **same** `cache line`
    * which is generally **beneficial** for `single-threaded` applications
- However, in a `multi-threaded` context, this can lead to performance degradation
- The issue arises because if `other threads` attempt to **lock** a `mutex` that is **already locked** by the `current thread`
    * This can lead to **cache ping-pong**, where the `mutex` constantly passes between CPU caches
        + because the `lock operation` is typically implemented as a `read-modify-write` atomic operation
    * The `data` residing in the **same** `cache line` as the `mutex` can be **negatively affected** by this contention, further degrading performance
- One common solution is to introduce **`padding`** to ensure that the `mutex` and `data` do **not** share the **same** `cache line`
    ```cpp
    // test the mutex contention issue
    struct Data {
        std::mutex m;
        char padding[std::hardware_destructive_interference_size];     
        MyData data;
    };

    // test for false sharing of array data
    struct Data {
        Item1 d1;
        Item2 d2;
        char padding[std::hardware_destructive_interference_size];
    };
    Data myArray[256]
    ```
    * you can test your class whether the **cache-line** was the problem of your performance loss by using this `padding`

### Two Sections of Execution
A `program` can be **divided** into **two** `sections`
- **`Serial Section`**
    * This is where **only one** thread performs useful work
    * This typically occurs when a thread **locks** a mutex, causing other threads to **block** until the mutex is released.
- **`Parallel Section`**
    * In this section, **all** available processors are utilized to **perform** useful work **simultaneously**
- To **maximize performance** on multi-core systems, it's important to
    * **Minimize** the size of the `serial section` 
        + or **reduce** the potential for threads to be **blocked**
    * In other words, **increasing** the `parallel section` and ensuring it remains well-fed with work can lead to significant performance gains

### Hiding Latency
It's important to recognize that there are situations where it's **inevitable** for a thread to `wait` for another thread
- In this case, the core concept is to **hide** this `latency` by giving the system useful **work** to do **during the waiting period**
    * For instance
    * If a thread is **blocked** while waiting for an `I/O operation` to complete
        * **asynchronous** `I/O` can be utilized, allowing the thread to perform other useful tasks while the `I/O operation` occurs in the **background**
    * or if a thread is waiting for another thread to complete an operation,
        * instead of blocking, the waiting thread could potentially perform the operation itself
        * This is seen in [**helping**](https://sadoe3.github.io/cpp/concurrency-chapter7/#example-1) techniques which we've implemented before

### Improving Responsiveness
The core principle is to **separate concerns**
- For example, in a graphical user interface (GUI) application
    * The `main GUI thread` handles the user interface
    * Multiple `event-handling threads` manage background tasks
- This separation allows the system to remain responsive by ensuring that the GUI thread isn't blocked by long-running operations, improving the overall user experience
    ```cpp
    std::vector<std::thread> taskThreads;       // managed well in a certain manner
    void doMainGUIThread() {
        while(true) {
            Event event=get_event();
            if(event.type==EventType::Quit)
                break;
            processEvent(event);
        }
    }
    void doEventHandlingThread(Event event) {
        while(event.isDone == false && event.isCancelled == false)
            performEventHandling(event);
        if(event.isCancelled)
            performCleanup();
        else
            postGUIEvent(event);                // let main gui thread know that the operation is done
    }
    void processEvent(const Event& event) {
        switch(event.type) {
            case EventType::START:
                taskThreads[properIndex]=std::thread(task, event);
                break;
            case EventType::STOP:
                event.isCancelled = true;
                taskThreads[properIndex].join();
                break;
            case EventType::DONE:
                taskThreads[properIndex].join();
                displayResult();
                break;
        }
    }
    ```


## Example
Now, let's dive into writing some **concurrent code**

### Parallel `find()`
```cpp
#include <thread>
#include <future>
#include <vector>

class ThreadsGuard {
public:
	explicit ThreadsGuard(std::vector<std::thread>& inputThreads) : threads(inputThreads) {}
	~ThreadsGuard() {
        for (auto& currentThread : threads) {
            if (currentThread.joinable())
                currentThread.join();
        }
	}

    ThreadsGuard(const ThreadsGuard&) = delete;
    ThreadsGuard& operator= (const ThreadsGuard&) = delete;
private:
	std::vector<std::thread> &threads;
};


template<typename Iterator, typename MatchType>
Iterator findParallel(Iterator first, Iterator last, MatchType match) {
    struct FindElement {
        void operator()(Iterator begin, Iterator end, MatchType match, std::promise<Iterator>* result, std::atomic<bool>* isDone) {
            try {
                for (; (begin != end) && !isDone->load(); ++begin) {
                    if (*begin == match) {
                        result->set_value(begin);
                        isDone->store(true);
                        return;
                    }
                }
            }
            catch (...) {
                try {
                    result->set_exception(std::current_exception());
                    isDone->store(true);
                }
                catch (...) {
                    // this block exists to handle cases where the promise is set more than once, which throws an exception
                    // this block is okay to be empty
                }
            }
        }
    };
    const unsigned long LENGTH = std::distance(first, last);
    if (!LENGTH)
        return last;


    const unsigned long MIN_ELEMENTS_PER_THREAD = 25;
    const unsigned long MAX_THREADS = (LENGTH + MIN_ELEMENTS_PER_THREAD - 1) / MIN_ELEMENTS_PER_THREAD;   // MIN_ELEMENTS_PER_THREAD - 1 is added to handle integer division
    const unsigned long HARDWARE_THREADS = std::thread::hardware_concurrency();
    const unsigned long NUMBER_OF_THREADS = std::min(HARDWARE_THREADS != 0 ? HARDWARE_THREADS : 2, MAX_THREADS);
    const unsigned long BLOCK_SIZE = LENGTH / NUMBER_OF_THREADS;
    std::promise<Iterator> result;
    std::atomic<bool> isDone(false);
    std::vector<std::thread> threads(NUMBER_OF_THREADS - 1);    // other threads; need to subtract 1 because of this thread
    
    {
        ThreadsGuard joiner(threads);
        Iterator blockStart = first;
        for (unsigned long i = 0; i < (NUMBER_OF_THREADS - 1); ++i) {
            Iterator blockEnd = blockStart;
            std::advance(blockEnd, BLOCK_SIZE);
            threads[i] = std::thread(FindElement(), blockStart, blockEnd, match, &result, &isDone);
            blockStart = blockEnd;
        }
        FindElement()(blockStart, last, match, &result, &isDone);      // search for remaining elements
    }
    if (!isDone.load())
        return last;

    return result.get_future().get();
}
```
```cpp
int main() {
    std::vector<unsigned> collection(300);
    for (unsigned value = 0, endValue = 300; value < endValue; value++)
        collection[value] = value;

    unsigned matchValue = 85;
    auto result = findParallel(collection.cbegin(), collection.cend(), matchValue);
    std::cout << matchValue;
    if (result == collection.cend())
        std::cout << " is not found" << std::endl;
    else
        std::cout << " is found : " << *result << std::endl;

    return 0;
}
/*
result
85 is found : 85
*/
```
- It's worth noting that
    * `atomic<bool>` flag is used to check whether the data is found or not from multiple threads
    * and `std::promise<Iterator>` is used to get the result




[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}