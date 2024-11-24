---
title: "C++ Concurrency : Data Sharing"

categories:
    - cpp

tags:
    - [C++, Programming Language, Concurrency, data sharing, data, mutex]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-23
---

# C++ Concurrency : Chapter 3

> 이 포스트는 `C++ Concurrency in Action (2nd ed.)`을 바탕으로 작성되었습니다.

## Data Sharing
While the ease with which **data** can be **shared** between `multiple threads` in a `single process` is often seen as a **benefit**, it can also pose a significant **drawback**
- Therefore, you need to learn how to treate them **safely** which is covered in this chapter

### Read-Only
As I mentioned earlier, if the shared data is `read-only`, then there's **no possibility** for problems to occur - it's always **safe**
- The **problems** happen all due to the consequences of `writing` data


## Race Condition
In concurrency, a `race condition` is anything where the **outcome** depends on the **relative ordering of execution** of operations on two or more **threads**
- The C++ Standard also defines the term `*data race*` to mean the specific type of `race condition` that arises because of **concurrent modification** to a `single object`
    * `data races` cause the **undefined behavior**
- It's worth noting that `race conditions` are **hard to find** and **hard to re-occur**
    * Because the **relatvie ordering** can be changed whenever you start the program or
    * the `debugger` affects the **timing** of the program, even if only slightly

### How to Aovid
There are multiple ways to avoid `race conditions`
- **Wrap you data structure with a protection mechanism**
    * The C++ Standard Library provides several of these mechanism, which are covered later in this post
- **Lock-free programming**
    * This option is to modify the **design** of your data structure and its invariants so that the **modifications** are done as a **series of indivisible changes**
    * This method is covered in later chapter
- **Software Transactional Memory**
    * STM tries to handle the **updates** to the `data structure` as a `transaction`, just as **updates** to a `database` are done within a `transaction`
    * Because it's an active **research** area, it won't be covered


## `std::mutex`
The most basic **mechanism** for **protecting share data** provided by the C++ Standard is to utlize `std::mutex`
- You can **lock** and **unlock** the data by using `std::mutex`
- Once a certain `thread` **locks** the data, it has the right to **modify** the data in a **mutually exclusive** way
    * which means that `other threads` that attempt to access that data have to **wait** until that thread finishes its modification
- It's worth noting that `other threads` **cannot** `read-only` the data because the access itself is blocked
    * In order to resolve this issue, you can use `std::shared_mutex` which is coverd later in this post    

### Where to Declare?
While a `std::mutex` object can be declared **locally** within a function or block of code, it is generally preferred to declare it at a **global** or **larger scope**
- This is because
    * The mutex needs to be **accessible** to any thread that requires synchronization with the shared resource
    * A mutex should **live** as long as the resource it protects
- If the **local** `std::mutex` object satisfies the these conditions above, it is **acceptable** to use a **local** `std::mutex` object

### Default Construction
It's worth noting that a `std::mutex` is **always** `default-constructed`
- This is because the `std::mutex` does **not directly** `own` or `bind` to the **data** it protects
- Instead, it serves as an **independent object** used to `lock` and `unlock` access to **shared data**

### Locking and Unlocking
- **`lock()`**
    * Invoking `lock()` allows a thread to enter the **critical section** and begin modifying shared resources
- **`unlock()`**
    * Invoking `unlock()` allows a thread to leave the **critical section**, releasing the lock so other threads can proceed
- It's worth noting that **blocking** (waiting) happens when a thread **attempts to lock** a `std::mutex` object that is **already locked** by another thread
    * Provided that `Thread 1` calls `lock()` **first** and `thread 2` calls `lock()` later
        + `Thread 2` will start **waiting** until `Thread 1` calls `unlock()` since the `std::mutex` object is **already locked** by `Thread 1`
    * The important point to notice is that if `Thread 2` does **not call** `lock()`
        + it will **not wait** for `Thread 1` to unlock the mutex
    * That is, **Blocking only occurs when a thread tries to lock a `std::mutex` object that is already locked by another thread**

### Different Mutex Objects
The concepts which we've covered so far are related to using **same** `std::mutex` object
- If **different** `std::mutex` objects are used, **blocking** would **not occur**
    * For instance, if `Thread 1` locks `Mutex 1` **first** and `Thread 2` locks `Mutex 2` later
    * `Thread 2` does **not wait** for `Thread 1` because `Mutex 2` is **not locked** (`Mutex 1` is locked)

### Recommended Practice: `std::lock_guard`
Instead of **directly calling** `lock()` and `unlock()`, it is **recommended** to use `std::lock_guard` (which uses the concept of `RAII`) 
- The `std::lock_guard` automatically `locks` the `std::mutex` object in its **constructor** and `unlocks` it in its **destructor**
    + This ensures the `std::mutex` object is properly released even if an **exception** occurs
    + which is similar to using `ThreadGuard` for threads to call `.join()`
- The example below shows basic usage of `std::lock_guard`
    ```cpp
    // function version
    void addElement(std::list<int>& collection, std::mutex& mutexCollection, int item) {
        // Automatically locks the mutex and unlocks it when the scope is exited
        std::lock_guard<std::mutex> guard(mutexCollection);
        collection.emplace_back(item);
    }

    // class version
    class DataWrapper {
    public:
        void doSomething() {
            std::lock_guard guard(m);
            // do something with data
        }
    private:
        DataType data;
        std::mutex m;
    };
    ```
- It's worth noting that `std::lock_guard` is designed for locking a **single** `std::mutex` object
    * If you want to lock **multiple** `std::mutex` objects **safely**, use `std::scoped_lock`

### Version Up your Compiler
If you **cannot** use `std::scoped_lock` which was introduced from `C++17`
- You need to **version up** your compiler so that it compiles the **modern C++ features**
- If you use `Visual Studio`, you can follow the steps below
    * Project Properties -> C/C++ -> Language
    * C++ Language Standard -> `C++ 17` or `C++ 20` -> Apply


## Race Condition Issues

### Handles to the Protected Data
It's important **not** to **return** the `handle to the protected data` or **pass** it to the `user-supplied functions` which are **not** in your **control**
- This is because these `backdoors` provide the oppportunity to **modify** the protected data **without locking**

### `std::stack`
Although `std::stack` is a well-made `container adapter`, there might be a situation where `race condition` occurs due to its **interface**
- Example case
    * The `empty()` method followed by the `top()` method introduces a potential `race condition`, as **another thread** may invoke `top()` **between these two calls**, making the `top()` operation **unavailable**.
- Therefore, you need to implement a `wrapper class` to add **thread-safety** to `std::stack`
    ```cpp
    #include <mutex>
    #include <memory>
    #include <stack>
    #include <exception>

    struct EmptyStack : std::exception {
        const char* what() const noexcept {
            std::exception::what();
        }
    };

    template <typename T>
    class ThreadSafeStack {
    public:
        ThreadSafeStack() {}
        ThreadSafeStack(const ThreadSafeStack& existingStack) {
            std::lock_guard<std::mutex> guard(m);
            data = existingStack.data;
        }
        ThreadSafeStack& operator=(const ThreadSafeStack&) = delete;

        void push(T newValue) {
            std::lock_guard<std::mutex> guard(m);
            data.push(std::move(newValue));
        }
        std::shared_ptr<T> pop() {
            std::lock_guard<std::mutex> guard(m);
            if (data.empty())
                throw EmptyStack();

            const std::shared_ptr<T> ptr = std::make_shared<T>(data.top());
            data.pop();
            return ptr;
        }
        void pop(T& value) {
            std::lock_guard<std::mutex> guard(m);
            if (data.empty())
                throw EmptyStack();

            value = data.top();
            data.pop();
        }
        bool empty() const {
            std::lock_guard<std::mutex> guard(m);
            return data.empty();
        }

    private:
        std::stack<T> data;
        std::mutex m;
    };
    ```
- With `std::shared_ptr<T> pop()`, you can safely **return** the `pointer` to the `popped item`
    * Which might not work for basic `pop()` method that returns the copied value of the item
    * because the **copy constructor** for the object on the `std::stack` can throw an **exception**
    * but making a pointer is free from this issue
- With `void pop(T&)`, you can safely **copy** the `popped item` to the given `parameter`
    * If you use the basic `pop()` method, there are **two copy operations** to occur
    * but this version does **copy** only **once**
    * hence if you want to **return** the `copied item` but **avoid** `additional copy`, use this method


## Deadlock

### Race Condition and Deadlock
- A **race condition** happens
    * when only a **single** `std::mutex` object is involved
- A **deadlock** happens 
    * when **multiple** `std::mutex` objects are involved, waiting for each other to release `std::mutex` objects

### Example Case
Suppose that `Thread A` holds `mutex A` and `Thread B` holds `mutex B`
- Now, `Thread A` wants to acquire `mutex B`, so it waits for `mutex B` to be released.
- Meanwhile, `Thread B` wants to acquire `mutex A`, so it waits for `mutex A` to be released.
- Then, the **deadlock** happens, leading to infinite waiting

### Simplest Solution: Locking Mutexes in the Same Order
To prevent deadlocks, one simple solution is to always **lock** and **unlock** the `std::mutex` objects in the **same order**
- However, this is **not** a **universal** solution
- For instance, consider a `swap` function
    * which always lock the first `std::mutex` parameter first, and the second parameter later (**fixed ordering**)
- What if two threads call the same `swap` function, but the **order** of parmeters is **reversed**?
    * now, deadlock happens

### How to Resolve It: Locking Mutexes at Once
Instead of locking mutexes one by one, you can **lock** and **unlock** all necessary `std::mutex` objects **simultaneously** to prevent deadlock, making the `order of locking` **trivial**
- The code below shows examples of this
    ```cpp
    class Data { /* definition */ };
    void swap(Data&, Data&) {
        // definition
    }

    class MyClass {
        friend void swap(MyClass& lhs, MyClass& rhs) {
            if (&lhs == &rhs) return;       // Avoid swapping the same object
            
            // Lock both mutexes simultaneously
            std::lock(lhs.m, rhs.m);        // Locks both mutexes without blocking
            // Adopt the existing locks to avoid re-locking
            std::lock_guard<std::mutex> guardLeft(lhs.m, std::adopt_lock);
            std::lock_guard<std::mutex> guardRight(rhs.m, std::adopt_lock);
        
            swap(lhs.data, rhs.data);
        }
    public:
        MyClass(const Data& inputData) : data(inputData) {}

    private:
        Data data;
        std::mutex m;
    };
    ```
    * `std::lock()` **locks** multiple `std::mutex` objects (can be more than two) **at once**
    * `std::lock_guard` is used to **automatically unlock** the `std::mutex` objects when they go out of scope
    * `std::adopt_lock` is passed to **inform** that those `std::mutex` objects are **already locked** so that `std::lock_guard` will not attempt to re-lock in its constructor
- To simplify the code above and improve safety, you can use `std::scoped_lock` introdued from `C++ 17` Standard
    ```cpp
    // same code

    friend void swap(MyClass& lhs, MyClass& rhs) {
        if (&lhs == &rhs) return; // Avoid swapping the same object
        
        // Locks both mutexes and ensures they are unlocked automatically when leaving scope
        std::scoped_lock guard(lhs.m, rhs.m);
        swap(lhs.data, rhs.data);
    }
    ```
    * `std::scoped_lock` only **locks** the `std::mutex` objects **at once** in its **constructor** but also **unlocks** them in its **destructor** 
    * This approach eliminates manual lock management, reduces complexity, and improves code readability.

### Other Possibilities
It's possible for deadlocks to happen even without `lock()` due to `join()` where one thread waits for the other, and same for the other thread
- Therefore, you need to keep in mind the **guidelines below**
- **Avoid holding multiple locks**
    * Do not acuqire a lock if you already hold one
    * If you need to lock **multiple** `std::mutex` objects,
        + lock them **at once**
- **Avoid calling user-supplied code while holding a lock**
    * User-supplied code can do anything, including attempt to **lock**
    * which **violates** the first guideline
- **Acquire locks in a fixed order**
    * If you need to lock **multiple** `std::mutex` objects, but you **cannot lock** them **simultaneously**
    * Try to **lock** them in the **same order** in `every thread`
- **Use a lock hierarchy**
    ```cpp
    // class implementation
    class HierarchicalMutex {
    public:
        explicit HierarchicalMutex(unsigned long value) : hierarchyValue(value), previousHierarchyValue(0) {}
        void lock() {
            checkForHierarchyViolation();
            internalMutex.lock();
            updateHierarchyValue();
        }
        void unlock() {
            if (thisThreadHierarchyValue != hierarchyValue)
                throw std::logic_error("mutex hierarchy violated");
            thisThreadHierarchyValue = previousHierarchyValue;
            internalMutex.unlock();
        }
        bool try_lock() {
            checkForHierarchyViolation();
            if (!internalMutex.try_lock())
                return false;
            updateHierarchyValue();
            return true;
        }

    private:	
        void checkForHierarchyViolation() {
            if (thisThreadHierarchyValue <= hierarchyValue)
                throw std::logic_error("mutex hierarchy violated");
        }
        void updateHierarchyValue() {
            previousHierarchyValue = thisThreadHierarchyValue;
            thisThreadHierarchyValue = hierarchyValue;
        }

        std::mutex internalMutex;
        unsigned long const hierarchyValue;
        unsigned long previousHierarchyValue;
        static thread_local unsigned long thisThreadHierarchyValue;
    };
    thread_local unsigned long HierarchicalMutex::thisThreadHierarchyValue(ULONG_MAX);
    ```
    ```cpp
    // basic use
    
    ```


### unique_lock




[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}