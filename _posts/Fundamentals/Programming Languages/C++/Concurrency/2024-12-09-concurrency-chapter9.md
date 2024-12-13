---
title: "C++ Concurrency : Advanced Thread Management"

categories:
    - cpp

tags:
    - [C++, Programming Language, Concurrency, thread, thread pool, thread interruption]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-12-09
---

# C++ Concurrency : Chapter 9

> 이 포스트는 `C++ Concurrency in Action (2nd ed.)`을 바탕으로 작성되었습니다.

## Thread Pool
Without a `thread pool`, you would need to **manually and explicitly** manage threads by creating `std::thread` objects for each individual task
- On most systems, it is **impractical** to create a separate thread for every task, especially when tasks can potentially be executed in parallel
- A **`thread pool`** is a mechanism that maintains a **fixed number of `worker threads`** (typically **equal** to the number returned by `std::thread::hardware_concurrency()`) 
    * which are responsible for processing tasks
- When tasks are **submitted** to the pool, they are placed on a `queue` of pending work
- Each task is then **taken** from the `queue` by one of the `worker threads`, which executes the task and then loops back to take another task from the `queue`

### No Waiting
The **simplest** way to implement a `thread pool` is to create one that has **no** mechanism for **waiting** for tasks to complete
```cpp
class ThreadPool {
public:
    ThreadPool() : isThreadPoolDone(false), joiner(threads) {
        const unsigned THREAD_COUNT = std::thread::hardware_concurrency();      // change this number as you want
        try {
            for (unsigned currentThread = 0; currentThread < THREAD_COUNT; ++currentThread)
                threads.push_back(std::thread(&ThreadPool::spawnWorkerThread, this));
        }
        catch (...) {
            isThreadPoolDone = true;
            throw;
        }
    }
    ~ThreadPool() {
        isThreadPoolDone = true;
    }

    template<typename FunctionType>
    void submit(FunctionType f) {
        workQueue.push(std::function<void()>(f));
    }

private:
    void spawnWorkerThread() {
        while (!isThreadPoolDone) {
            std::function<void()> task;
            if (workQueue.tryPop(task))
                task();
            else
                std::this_thread::yield();
        }
    }

    std::atomic<bool> isThreadPoolDone;
    QueueThreadSafe<std::function<void()> > workQueue;
    std::vector<std::thread> threads;
    ThreadsGuard joiner;
};
```
- This example utilized the [**`QueueThreadSafe`**](https://sadoe3.github.io/cpp/concurrency-chapter6/#a-thread-safe-queue) which we've implemented before
- You can adjust `THREAD_COUNT` based on your current requirements
- It's important to note that `isThreadPoolDone` and `workQueue` must be **declared before** `threads`, and `threads` must be **declared before** `joiner`
    * This ensures that the members are **destroyed** in the **correct order**
- If certain `workerThreads` have no work to do, `std::this_thread::yield()` is called
    * This **hints** to the OS's `thread scheduler` that the current thread is willing to give up its remaining time slice, allowing **other threads to run**
    * However, it is crucial to understand that this does **not** guarantee the thread will be **blocked** or immediately preempted
        + It simply informs the scheduler that it is acceptable for other threads to execute if they are ready
    * In other words, it does **not explicitly pause** the thread
        + rather, it **gives** the scheduler the **option** to schedule another thread if needed
- Since we do **not wait** for tasks to complete in this case, there is **no return value**
    * If you need to block or return a value, you will **need** to implement a more complex version that supports **waiting**

### Waiting for Tasks
In order to implement **waiting** feature, `std::condition_variable` (waiting only) or `std::future<>` (returning a value) is needed
```cpp
class FunctionWrapper {
    struct ImplementationBase {
        virtual void call() = 0;
        virtual ~ImplementationBase() {}
    };
    // polymorphism is required 
    template<typename F>
    struct ImplementationType : ImplementationBase {
        F f;
        ImplementationType(F&& inputFunctor) : f(std::move(inputFunctor)) {}
        void call() { 
            f();
        }
    };

public:
    FunctionWrapper() = default;
    FunctionWrapper(FunctionWrapper&& other) : impl(std::move(other.impl)) {}
    // move the packaged_task
    template<typename F>
    FunctionWrapper(F&& f) : impl(new ImplementationType<F>(std::move(f))) {}
    FunctionWrapper(const FunctionWrapper&) = delete;
    FunctionWrapper(FunctionWrapper&) = delete;

    FunctionWrapper& operator=(FunctionWrapper&& other) {
        impl = std::move(other.impl);
        return *this;
    }
    FunctionWrapper& operator=(const FunctionWrapper&) = delete;

    void operator()() {
        impl->call();
    }


private:
    std::unique_ptr<ImplementationBase> impl;
};
```
```cpp
class ThreadPool {
public:
    ThreadPool() : isThreadPoolDone(false), joiner(threads) {
        const unsigned THREAD_COUNT = std::thread::hardware_concurrency();
        try {
            for (unsigned currentThread = 0; currentThread < THREAD_COUNT; ++currentThread)
                threads.push_back(std::thread(&ThreadPool::spawnWorkerThread, this));
        }
        catch (...) {
            isThreadPoolDone = true;
            throw;
        }
    }
    ~ThreadPool() {
        isThreadPoolDone = true;
    }

    template<typename FunctionType>
    std::future<std::invoke_result_t<FunctionType>> submit(FunctionType f) {
        std::packaged_task<std::invoke_result_t<FunctionType>()> task(std::move(f));
        std::future<std::invoke_result_t<FunctionType>> result(task.get_future());
        workQueue.push(std::move(task));
        return result;
    }

private:
    void spawnWorkerThread() {
        while (!isThreadPoolDone) {
            FunctionWrapper task;
            if (workQueue.tryPop(task))
                task();
            else
                std::this_thread::yield();
        }
    }

    std::atomic<bool> isThreadPoolDone;
    QueueThreadSafe<FunctionWrapper> workQueue;
    std::vector<std::thread> threads;
    ThreadsGuard joiner;
};
```
- It's worth noting that in C++20, `typename std::result_of<FunctionType()>::type` (used in the textbook) was completely **removed**
    * You should now use `std::invoke_result_t<FunctionType>` instead
- The main change here is to replace `std::function<void()>` with the `FunctionWrapper` class
    * This ensures that the task is **moved** to a separate thread from the queue
        + This is crucial because `std::packaged_task<>` is **not copyable**
    * **Polymorphism** is needed because the calling thread does **not** need to **know** the `return type` of the functor

### Example: `accumulateParallel()`
```cpp
template <typename Iterator, typename T>
T accumulateBlock(Iterator first, Iterator last) {
    Iterator currentBlock = first;
    T result = *currentBlock;
    std::advance(currentBlock, 1);

    while (currentBlock != last) {
        result += *currentBlock;
        std::advance(currentBlock, 1);
    }
    return result;
}

template <typename Iterator, typename T>
T accumulateParallel(Iterator first, Iterator last, const T &init) {
    const unsigned long LENGTH = std::distance(first, last);
    if (!LENGTH)
        return init;

    unsigned long const BLOCK_SIZE = 25;        // need to set proper BLOCK_SIZE
    unsigned long const NUMBER_OF_BLOCKS = (LENGTH + BLOCK_SIZE - 1) / BLOCK_SIZE;
    std::vector<std::future<T>> results(NUMBER_OF_BLOCKS - 1);
    ThreadPool pool;
    Iterator blockStart = first;
    for (unsigned long currentBlock = 0, endBlock = NUMBER_OF_BLOCKS - 1; currentBlock < endBlock; ++currentBlock) {
        Iterator blockEnd = blockStart;
        std::advance(blockEnd, BLOCK_SIZE);
        results[currentBlock] = pool.submit([=] { return accumulateBlock<Iterator, T>(blockStart, blockEnd); } );
        blockStart = blockEnd;
    }
    T lastResult = accumulateBlock<Iterator, T>(blockStart, last);   // accumulate last block

    T result = init;
    for (unsigned long currentBlock = 0, endBlock = NUMBER_OF_BLOCKS - 1; currentBlock < endBlock; ++currentBlock)
        result += results[currentBlock].get();
    result += lastResult;
    return result;
}

int main() {
    std::vector<unsigned> collection(300);
    for (unsigned value = 0, endValue = 300; value < endValue; value++)
        collection[value] = value;

    unsigned result = 0;
    result = accumulateParallel(collection.cbegin(), collection.cend(), result);
    std::cout << result << std::endl;

    return 0;
}
/*
result
11175
*/
```
- Choosing the **proper** `BLOCK_SIZE` is **crucial** when performing concurrent work
    * If the it is **too small**, the code may run **more slowly** with a `thread pool` than with a `single thread`
    * This is because there is inherent **overhead** in
        + submitting a task to the thread pool,
        + having a worker thread execute it,
        + and passing the return value through `std::future<>`

### Hiding Latency
As we've learned from the previous [**chapter**](https://sadoe3.github.io/cpp/concurrency-chapter8/#hiding-latency), it's recommended to **hide waiting** to increase performance
```cpp
void ThreadPool::runPendingWork() {
    FunctionWrapper task;
    if (workQueue.tryPop(task))
        task();
    else
        std::this_thread::yield();
}
```
- In a `thread pool`, this can be achieved by implementing `runPendingTask()`
    * This function is **called** when **blocking** is required
    * You can check whether **blocking** is needed by calling `futureObject.wait_for(std::chrono::seconds(0))`
        + If it returns `std::future_status::timeout`, then the task needs to **block**

### Example : `quickSortParallel()`
Let's reuse the [**quickSortParallel**](https://sadoe3.github.io/cpp/concurrency-chapter4/#parallel-quicksort) that we've implemented before to apply **hiding latency** feature
```cpp
template<typename T>
struct QuickSorter {
    ThreadPool pool;

    std::list<T> doSort(std::list<T>& input) {
        if (input.empty())
            return input;

        std::list<T> result;
        result.splice(result.begin(), input, input.begin());
        const T& pivot = *result.begin();

        auto dividePoint = std::partition(input.begin(), input.end(), [&](const T& val) {return val < pivot; });
        std::list<T> lowerPart;
        lowerPart.splice(lowerPart.end(), input, input.begin(), dividePoint);
        std::future<std::list<T> > sortedLower = pool.submit(std::bind(&QuickSorter::doSort, this, std::move(lowerPart)));
        std::list<T> sortedHigher (doSort(input));

        result.splice(result.end(), sortedHigher);
        while (sortedLower.wait_for(std::chrono::seconds(0)) == std::future_status::timeout)
            pool.runPendingWork();    // hiding latency!
        result.splice(result.begin(), sortedLower.get());
        return result;
    }
};

template<typename T>
std::list<T> quickSortParallel(std::list<T> input) {
    if (input.empty())
        return input;
    QuickSorter<T> sorter;
    return sorter.doSort(input);
}
```
```cpp
int main() {
    std::list<int> collection{ 1, 42, -23, 5, 7, 40, 2, 3, 6, 29, 10, 17, 11, -12, 13, -14, 15, 0, 50 };
    std::cout << "unsorted" << std::endl;
    for (auto element : collection)
        std::cout << element << " ";
    std::cout << std::endl;


    collection = quickSortParallel(std::move(collection));
    std::cout << "sorted" << std::endl;
    for (auto element : collection)
        std::cout << element << " ";
    std::cout << std::endl;

    return 0;
}

/*
result

unsorted
1 42 -23 5 7 40 2 3 6 29 10 17 11 -12 13 -14 15 0 50
sorted
-23 -14 -12 0 1 2 3 5 6 7 10 11 13 15 17 29 40 42 50
*/
```

### Avoiding Contention
So far, there is only a **single** `work queue` that is accessed by **multiple** `threads`
- This can lead to **high contention** on the `work queue`, which may **degrade performance**
- To resolve this issue, you should provide an **additional** `local queue` for **each** `thread`, ensuring that tasks originating from a particular thread are handled by that thread
    * Each thread **first** attempts to perform tasks from its own `local queue`
    * If its `local queue` is **empty**, the thread will then attempt to perform tasks from the `global queue`
- The code below demonstrates a simple implementation of this technique
    ```cpp
    class ThreadPool {
    public:
        ThreadPool() : isThreadPoolDone(false), joiner(threads) {
            const unsigned THREAD_COUNT = std::thread::hardware_concurrency();
            try {
                for (unsigned currentThread = 0; currentThread < THREAD_COUNT; ++currentThread)
                    threads.push_back(std::thread(&ThreadPool::spawnWorkerThread, this));
            }
            catch (...) {
                isThreadPoolDone = true;
                throw;
            }
        }
        ~ThreadPool() {
            isThreadPoolDone = true;
        }

        template<typename FunctionType>
        std::future<std::invoke_result_t<FunctionType>> submit(FunctionType f) {
            std::packaged_task<std::invoke_result_t<FunctionType>()> task(std::move(f));
            std::future<std::invoke_result_t<FunctionType>> result(task.get_future());

            if (workQueueLocal == nullptr)
                workQueueGlobal.push(std::move(task));
            else
                workQueueLocal->push(std::move(task));

            return result;
        }
        void runPendingWork() {
            FunctionWrapper task;
            if (workQueueLocal && !workQueueLocal->empty()) {
                task = std::move(workQueueLocal->front());
                workQueueLocal->pop();
                task();
            }
            else if (workQueueGlobal.tryPop(task))
                task();
            else
                std::this_thread::yield();
        }

    private:
        void spawnWorkerThread() {
            workQueueLocal.reset(new std::queue<FunctionWrapper>);

            while (!isThreadPoolDone)
                runPendingWork();
        }

        std::atomic<bool> isThreadPoolDone;
        QueueThreadSafe<FunctionWrapper> workQueueGlobal;
        static thread_local std::unique_ptr<std::queue<FunctionWrapper>> workQueueLocal;    // local work queue
        std::vector<std::thread> threads;
        ThreadsGuard joiner;
    };
    ```
- This approach reduces contention
- However, if the **distribution** of work is **uneven**, it can result in one thread having a lot of work in its queue while others have none
    * To address this issue, you should allow threads to **`steal`** work from each other's queues
    * If a thread's `local queue` is **empty** and the `global queue` has **no work**, the thread can `steal` tasks from `another thread's queue`

### Stealing Work
The code below demonstrates a simple implementation of a **lock-based** `queue` that supports **`work stealing`**
```cpp
class QueueWorkStealing {
public:
    QueueWorkStealing() {}
    QueueWorkStealing(const QueueWorkStealing& other) = delete;
    QueueWorkStealing& operator=(const QueueWorkStealing& other) = delete;

    void push(FunctionWrapper data) {
        std::lock_guard<std::mutex> lock(m);
        myQueue.push_front(std::move(data));
    }
    bool tryPop(FunctionWrapper& result) {
        if (empty())
            return false;

        std::lock_guard<std::mutex> lock(m);
        result = std::move(myQueue.front());
        myQueue.pop_front();
        return true;
    }
    // stealing work!!
    bool trySteal(FunctionWrapper& result) {
        if (empty())
            return false;

        std::lock_guard<std::mutex> lock(m);
        result = std::move(myQueue.back());
        myQueue.pop_back();
        return true;
    };
    bool empty() const {
        std::lock_guard<std::mutex> lock(m);
        return myQueue.empty();
    }

private:
    std::deque<FunctionWrapper> myQueue;
    mutable std::mutex m;
};
```
```cpp
// updated ThreadPool
class ThreadPool {
public:
    ThreadPool() : isThreadPoolDone(false), joiner(threads) {
        const unsigned THREAD_COUNT = std::thread::hardware_concurrency();
        try {
            for (unsigned currentThread = 0; currentThread < THREAD_COUNT; ++currentThread)
                localQueues.push_back(std::unique_ptr<QueueWorkStealing>(new QueueWorkStealing));

            for (unsigned currentThread = 0; currentThread < THREAD_COUNT; ++currentThread)
                threads.push_back(std::thread(&ThreadPool::spawnWorkerThread, this, currentThread));
        }
        catch (...) {
            isThreadPoolDone = true;
            throw;
        }
    }
    ~ThreadPool() {
        isThreadPoolDone = true;
    }

    template<typename FunctionType>
    std::future<std::invoke_result_t<FunctionType>> submit(FunctionType f) {
        std::packaged_task<std::invoke_result_t<FunctionType>()> task(std::move(f));
        std::future<std::invoke_result_t<FunctionType>> result(task.get_future());

        if (workQueueLocal == nullptr)
            workQueueGlobal.push(std::move(task));
        else
            workQueueLocal->push(std::move(task));

        return result;
    }
    void runPendingWork() {
        FunctionWrapper task;
        // priority: 1. local queue 2. global queue 3. steal task from other local queue
        if(popTaskFromLocalQueue(task) || popTaskFromGlobalQueue(task) || popTaskFromOtherLocalQueue(task))
            task();
        else
            std::this_thread::yield();
    }

private:
    void spawnWorkerThread(unsigned inputIndex) {
        threadIndex = inputIndex;
        workQueueLocal = localQueues[threadIndex].get();
        while (!isThreadPoolDone)
            runPendingWork();
    }
    bool popTaskFromLocalQueue(FunctionWrapper& task) {
        return workQueueLocal && workQueueLocal->tryPop(task);
    }
    bool popTaskFromGlobalQueue(FunctionWrapper& task) {
        return workQueueGlobal.tryPop(task);
    }
    bool popTaskFromOtherLocalQueue(FunctionWrapper& task) {
        for (unsigned currentQueueIndex = 0, endIndex = localQueues.size(); currentQueueIndex < endIndex; ++currentQueueIndex) {
            const unsigned INDEX = (threadIndex + currentQueueIndex + 1) % localQueues.size();
            if (localQueues[INDEX]->trySteal(task))
                return true;
        }
        return false;
    }


    std::atomic<bool> isThreadPoolDone;
    QueueThreadSafe<FunctionWrapper> workQueueGlobal;
    std::vector<std::unique_ptr<QueueWorkStealing>> localQueues;            // manage local localQueues at once
    std::vector<std::thread> threads;
    ThreadsGuard joiner;
    static thread_local QueueWorkStealing* workQueueLocal;                  // local work queue
    static thread_local unsigned threadIndex;
};
// define static members
thread_local unsigned ThreadPool::threadIndex = 0;
thread_local QueueWorkStealing* ThreadPool::workQueueLocal = nullptr;
```
- The `steal` functionality is straightforward
    * If there is **no work** in the current thread's `local queue` and the `global queue`,
    * `Steal` a task from one of the `other local queues`
        + `Steal` the **last task** by calling `.back()` instead of `.front()`
- Other mechanics **remain** as they were
- It is also possible to implement a `queue` that supports **work stealing** in a **lock-free** manner, if desired.


## Thread Interruption
In many sitautions it's desirable for one thread to signal another thread to stop

### Basic Implementation
If you want the `interruption` only, then `std::atomic<bool>` at **global scope** is enough for the implementation
```cpp
std::atomic<bool> flagInterrupt(false);
// A function that checks the interruption flag
void do_work() {
    while (true) {
        // Check the atomic interrupt flag periodically
        if (flagInterrupt.load(std::memory_order_acquire)) {
            std::cout << "Thread interrupted, stopping work...\n";
            break;
        }

        // Simulate some work
        std::cout << "Working...\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
    }
}

int main() {
    std::thread worker(do_work);
    std::this_thread::sleep_for(std::chrono::seconds(2));

    // Set the interrupt flag to signal the worker thread to stop
    flagInterrupt.store(true, std::memory_order_release);

    worker.join();
    return 0;
}
```
- If you want more, you need to implement a `wrapper class`

### Refined Implementation
The main idea is just to add `.interrupt()` method to the original `std::thread` class
```cpp
// custom exception class for the interruption
class InterruptException : public std::exception {
public:
    InterruptException() = default;
};
```
```cpp
// wrapper class which supports the interruption
class ThreadInterruptible {
    friend void checkInterruption();
public:
    template<typename FunctionType, typename... Args>
    ThreadInterruptible(FunctionType f, const Args&... rest) : flag(nullptr) {
        std::promise<std::atomic<bool>*> p;                             // needed to get the value from the different thread
        threadInternal = std::thread([f, &p, &rest...] {
            p.set_value(&thisThreadInterruptFlag);                      // set the thread_local flag on separate thread to std::promise on current thread
            try {
                f(rest...);
            }
            catch (const InterruptException&) {
                std::cout << "interrupted" << '\n';                     // do something here; or just swallow it to terminate only
            }
        });

        std::this_thread::sleep_for(std::chrono::milliseconds(1));      // sleep for the case where immediate interrupt might occur until flag is properly reset
        flag = p.get_future().get();                                    // get the address of the thread_local flag on separate thread through std::future
    }

    // new functionality!!
    void interrupt() {
        if(flag)
            flag->store(true, std::memory_order_relaxed);
    }

    void join() {
        threadInternal.join();
    }
    void detach() {
        threadInternal.detach();
    }
    bool joinable() const {
        return threadInternal.joinable();
    }

private:
    static thread_local std::atomic<bool> thisThreadInterruptFlag;

    std::thread threadInternal;
    std::atomic<bool>* flag;
};
thread_local std::atomic<bool> ThreadInterruptible::thisThreadInterruptFlag = false;         // reset the flag to false when starting the new thread
```
```cpp
// function to check whether current thread is interrupted or not
void checkInterruption() {
    if (ThreadInterruptible::thisThreadInterruptFlag.load(std::memory_order_relaxed))
        throw InterruptException();
}
```
- But as you can see, there're some **techniques** involved to add the `.interrupt()`
- The code below shows the example of using `ThreadInterruptible`
    ```cpp
    void doSomething(int id) {
        while (true) {
            checkInterruption();
            std::cout << "working on " << id << '\n';
            std::this_thread::sleep_for(std::chrono::milliseconds(500));
        }
    }

    int main() {
        std::vector<ThreadInterruptible> threads;
        threads.push_back(ThreadInterruptible(doSomething, 1));
        threads.push_back(ThreadInterruptible(doSomething, 2));


        // interrupt all threads to stop!!
        for (unsigned i = 0; i < threads.size(); ++i)
            threads[i].interrupt();
        
        
        for (unsigned i = 0; i < threads.size(); ++i)
            threads[i].join();

        return 0;
    }
    /*
    result
    working on 1
    working on 2
    interrupted
    interrupted
    */
    ```
- It's also possible to add a functionality to **prevent** a certain thread from being **interrupted** if you want


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}