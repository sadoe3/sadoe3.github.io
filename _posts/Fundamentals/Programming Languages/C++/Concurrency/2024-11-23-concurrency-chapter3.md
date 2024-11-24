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
    HierarchicalMutex highMutex(10000);
    HierarchicalMutex lowMutex(5000);
    HierarchicalMutex otherMutex(6000);
    void doLowStuff() {
        std::lock_guard<HierarchicalMutex> guard(lowMutex);
        std::cout << "doing low level stuff" << std::endl;
    }
    void doHighStuff() {
        std::lock_guard<HierarchicalMutex> guard(highMutex);
        doLowStuff();
        std::cout << "doing high level stuff" << std::endl;

    }
    void threadA() {
        doHighStuff();
    }

    void doOtherStuff() {
        try {
            doHighStuff();
        }
        catch (const std::logic_error& e) {
            std::cerr << e.what() << std::endl;
            return;
        }

        std::cout << "doing other stuff" << std::endl;
    }
    void threadB() {
        std::lock_guard<HierarchicalMutex> guard(otherMutex);
        doOtherStuff();
    }

    // threadA
	std::thread t1(threadA);		// works properly
	t1.join();
    /* print result
    doing low level stuff
    doing high level stuff
    */

    // threadB
	std::thread t2(threadB);		// failed to call doHighStuff()
	t2.join();
    /* print result
    mutex hierarchy violated
    */
    ```
    * The core concept of this guideline is to divide your application into **layers** and identify all the `std::mutex` objects that may be locked in any given layer
    * When code tries to **lock** a `std::mutex` object, it is **not permitted** to **lock** that object
        + if the `hierarchy value` of that `std::mutex` object is **greater than or equal to** the value of the `std::mutex` object **already locked** in the lower layer
    * With `HierarchicalMutex` objects, it's **impossible** for **deadlocks** to occur
        + because the `HierarchicalMutex` objects themselves enforce the **lock ordering**
    * Although this is a common pattern, C++ Standard Library doesn't provide direct support
        * hence, you need to implement your own version like the example above
- **Extend these guidelines beyond locks**
    * As mentioned above, `.join()` can cause deadlocks
    * Therefore, it's highly recommended to apply these guidlines to `.join()` or other stuff which might lead to a **wait cycle**
        + For instance, avoid calling `.join()` if you already have a lock


## `std::unique_lock`
`std::unique_lock` is **similar** to `std::lock_guard` in that it supports the `RAII` idiom
- However, it offers more **flexibility**:
    * You **can** call `lock()` and `unlock()` **directly**, which is **not possible** with `std::lock_guard`
    * `std::unique_lock` allows you to **defer** the timing of when the `std::mutex` is **locked**
        + However, this comes with a **cost**, as it requires managing an `internal flag`, which may impact **performance** and **memory** usage
    * The **ownership** of the `std::mutex` can be **transferred** between `std::unique_lock` objects

### Deferred Locking
In the case of `std::lock_guard`, the `std::mutex` is **locked** at the time of **construction**
- With `std::unique_lock`, you can **defer** the timing of **locking** to whenever you want
    ```cpp
    std::mutex m;
    std::unique_lock<std::mutex> lockA(m, std::defer_lock);
    // m is not locked during construction
    // You can later call lock() to lock m at the appropriate time
    ```
- It's worth noting that if you do **not** provide `std::defer_lock`
    * the **constructor** of `std::unique_lock` will call `lock()`

### Ownership 
A `std::unique_lock` object **only owns** the `std::mutex` **if** `lock()` is **called** on it
- Until then, it does **not own** the `std::mutex` and will **not** call `unlock()` in its **destructor**
- If the `std::mutex` is **owned** by calling `lock()`
    * `unlock()` will be **called** automatically when the `std::unique_lock` is **destroyed**
- You can check whether a `std::unique_lock` **owns** the `std::mutex` by calling the `.owns_lock()` member function

### Transferring Ownership
The title is somewhat **misleading** because it's still **possible** to **transfer** `ownership` even if the `std::unique_lock` does **not own** the `std::mutex` object
- What actually happens is that the **right** to **manage** the `std::mutex` is **transferred** to another `std::unique_lock` object
- `Ownership` of a `std::unique_lock` can be **transferred** by **moving** it
    + just like **transferring** `ownership` of a `std::thread`
- The code below shows the basic example of this
    ```cpp
    std::mutex m;
    std::unique_lock<std::mutex> u1(m, std::defer_lock);
    std::unique_lock<std::mutex> u2 = std::move(u1);  // Ownership transferred to u2

    u1.lock();  // Undefined behavior! u1 no longer owns m
    u2.lock();  // Correct usage: u2 owns m
    ```
    ```cpp
    std::mutex m;
    std::unique_lock<std::mutex> f() {
        std::unique_lock<std::mutex> tempGuard(m, std::defer_lock);
        return tempGuard;
    }

    int main() {
        std::unique_lock<std::mutex> guard(f());
        guard.lock();  // Locking m after it's moved to guard from tempGuard
    }
    ```
- The **destructor** of a **moved-from** `std::unique_lock` does **not call** `unlock()`
    * because it no longer has a `std::mutex` object to manage
- If you try to call `lock()` on a **moved-from** `std::unique_lock` object
    * it results in **undefined behavior**
- You can use `std::lock` to **lock** multiple `std::unique_lock` objects **at once**
    ```cpp
    // same swap example
    friend void swap(MyClass& lhs, MyClass& rhs) {
        if (&lhs == &rhs) return;
        
        // same line count, but different order
        std::unique_lock<std::mutex> guardLeft(lhs.m, std::defer_lock);
        std::unique_lock<std::mutex> guardRight(rhs.m, std::defer_lock);
        std::lock(lhs.m, rhs.m); 
        
        swap(lhs.data, rhs.data);
    }
    ```

### Minimizing Lock Duration for Performance
In general, a **lock** should be held for the **minimum time** necessary to perform the required operations in order to optimize **performance**  
- This means you should **lock** a `std::mutex` object **only while** `accessing the shared data`
    * and perform any `data processing` **outside** the lock
- Therefore, **avoid** performing `time-consuming activities` while holding a **lock**
    ```cpp
    std::mutex m;
    std::unique_lock<std::mutex> myLock(m);
    Data inputData = readData();                        // Read data
    myLock.unlock();  

    ResultType result = processData(inputData);         // Time-consuming processing

    myLock.lock();    
    writeResult(inputData, result);                     // Write results
    ```


## Extra Details

### `std::call_once` and `std::once_flag`
In situations where multiple functions need to initialize some shared resource and can be run in separate threads
- We often face the challenge of ensuring that the `initialization` occurs **only once**, regardless of which thread calls it first
- In order to achieve this, you can use a combination of `std::call_once` and `std::once_flag` to ensure that a function is executed **only once**
    * no matter how many threads attempt to call it
- In the following example, `std::call_once()` ensures that the `openConnection()` is called **only once**, even if `sendData` or `receiveData` are called by **multiple threads**
    ```cpp
    class ConnectionManager {
    public:
        void sendData(DataPacket const& data) {
            std::call_once(initFlag, &ConnectionManager::openConnection, this);
            connection.sendData(data);
        }
        DataPacket receiveData() {
            std::call_once(initFlag, &ConnectionManager::openConnection, this);
            return connection.receiveData();
        }
        // Other member functions
    private:
        void openConnection() {
            // Code to initialize the connection
        }
        ConnectionHandle connection;
        std::once_flag initFlag;
        // Other data members
    };
    ```
    * The `std::call_once()` is used to ensure that `openConnection` is called only once, regardless of how many threads invoke `sendData` or `receiveData`
        + In order to make this possible, you need to pass `std::once_flag` object as the flag for checking
    * It's worth noting that when passing a `member function` to `std::call_once`, you must also pass the `this` pointer, as member functions **implicitly require** it
- In other words, if you're calling a `global function` (rather than a `member function`), you **don't need** to pass the `this` pointer
    ```cpp
    std::once_flag flag;
    void f() {
        // Code to execute once
    }
    std::call_once(flag, f);
    ```
    * `f()` will be called only once, regardless of how many times `std::call_once` is invoked across different threads

### `std::shared_mutex`
`std::shared_mutex` and `std::shared_timed_mutex` both provide a mechanism for **concurrent** `read/write` locking
- `std::shared_timed_mutex` supports `additional operations` which will be covered in the next chapter
    * If you don't need these extra features
    * it is recommended to use `std::shared_mutex` for better **performance**
- **Exclusive Ownership**
    * You can use `std::lock_guard`, `std::unique_lock`, or `std::scoped_lock` with a `std::shared_mutex` for **exclusive** access
- **Shared Ownership**
    * You can use `std::shared_lock<std::shared_mutex>` for **shared** access
- **Locking Behavior**
    * Multiple `std::shared_lock` objects on the **same** `std::shared_mutex` do **not block each other**
        + If one `std::shared_lock` is **already** holding the **lock**, other `std::shared_lock` objects **can** still acquire the **lock without waiting**.
    * When it comes to `std::unique_lock`, `std::lock_guard` and `std::scoped_lock`, these types **block** if any `std::shared_lock` objects are **already** holding the **lock**
        + They require **exclusive ownership**, so if the `std::shared_mutex` is held, they must **wait** until it is **released**
    * If a `std::unique_lock`, `std::lock_guard`, or `std::scoped_lock` acquires the **lock first**
        + `std::shared_lock` objects must **wait** for it to be **released** because the `std::shared_mutex` is **held exclusively**
- It's worth noting that, technically speaking, it is possible to use `std::shared_lock` for **concurrent** `writing` 
    * However, it is **not recommended** due to the risk of `data inconsistency` and `concurrency issues`
    * Therefore, try to use
        + `std::shared_lock<std::shared_mutex>` for `read` operations only
        + and others (`std::lock_guard`, `std::unique_lock`, or `std::scoped_lock`) for `write` operations
- `std::mutex` requires exclusive ownership for both `reading` and `writing`
    * Only one thread can hold the lock at a time regardless of whether it tries to read or write
- `std::shared_mutex` allows
    * **shared ownership** for `reading`
        + (**multiple** threads **can** acquire the **lock simultaneously** for `reading`)
    * and **exclusive ownership** for `writing`
    * Which means multiple threads can read concurrently, but only one thread can write at a time
- In order to utilize `std::shared_mutex` or `std::shared_lock`
    * include `<shared_mutex>` header file
- The code below shows an example of how `std::shared_mutex` works
    ```cpp
    #include <iostream>
    #include <shared_mutex>
    #include <thread>

    std::shared_mutex sharedMutex;
    void reader(int id) {
        std::shared_lock<std::shared_mutex> lock(sharedMutex);
        std::cout << "Reader " << id << " is reading\n";
    }
    void writer() {
        std::unique_lock<std::shared_mutex> lock(sharedMutex);
        std::cout << "Writer is writing\n";
    }

    int main() {
        std::thread t1(reader, 1);
        std::thread t2(reader, 2);
        std::thread t3(writer);

        t1.join();
        t2.join();
        t3.join();

        return 0;
    }
    /* possible output -> in this case, writer is called first but readers can be called first as well
    Writer is writing
    Reader Reader 1 is reading
    2 is reading
    */
    ```
    * **`Reader` threads** can run concurrently, as `std::shared_lock` allows multiple threads to hold the lock simultaneously for reading
    * The **`Writer` thread** will wait until all `Reader` threads release their shared locks
        + Once the writer acquires the lock in exclusive mode, no other threads can acquire it until the `writer` is done

### `std::recursive_mutex`
A `std::recursive_mutex` is similar to a normal `std::mutex`, but it **allows** `a thread` to call `lock()` **multiple times** on the **same** `std::recursive_mutex` object without causing a deadlock
- This makes it useful when a thread needs to acquire the same mutex repeatedly (e.g., in `recursive functions`)
- It's worth noting that if you **lock** the `std::recursive_mutex` object **multiple times** (e.g., `3 times`)
    * you must **unlock** it the **same number of times** (i.e., `3 times`) before the `std::recursive_mutex` object can be **locked** by `another thread`
    * In this case, `std::lock_guard<std::recursive_mutex>` or `std::unique_lock<std::recursive_mutex>` can be used to automatically handle **unlocking** when the it goes out of scope, ensuring that the `std::recursive_mutex` object is released properly
- If it's **absolutely necessary** to use `std::recursive_mutex` 
    * for example, when dealing with `recursive functions` that need to **lock** the **same** `std::recursive_mutex` object **multiple times**
    * then it's okay to use it
- Otherwise, try to **avoid** `std::recursive_mutex`
    * It's generally a good idea to reconsider your design to eliminate the need for it
    * A better design may avoid having a thread lock the same mutex multiple times, which can improve clarity and reduce the risk of mistakes


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}