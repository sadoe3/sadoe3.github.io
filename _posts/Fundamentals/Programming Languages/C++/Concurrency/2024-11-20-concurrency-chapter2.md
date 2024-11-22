---
title: "C++ Concurrency : Threads Management"

categories:
    - cpp

tags:
    - [C++, Programming Language, Concurrency, Threads Management, thread, RAII, exception, ownership, move]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-20
---

# C++ Concurrency : Chapter 2

> 이 포스트는 `C++ Concurrency in Action (2nd ed.)`을 바탕으로 작성되었습니다.

## `<thread>`
To utilize functions and classes for managing threads
- You must include the `<thread>` header file.

### `main()`
Every C++ program begins with **at least one thread**, which is the one running the `main()` function.
- After this initial thread is launched,
- You can create **additional threads**, each with its own **entry point function**.
- A thread **exits** when its entry point function returns.


## Construction of `std::thread`
To start a thread in C++, you must construct a `std::thread` object with an **initial entry function** (commonly referred to as the **thread function**).
- This entry function can be any **callable object**, such as a lambda, function pointer, or functor.

### Basic Mechanism
Creating a `std::thread` immediately starts the specified callable in a **separate thread**.
- The thread terminates when its entry function returns.

### What If No Initial Function is Specified?
If you do not specify an initial function:
- The **default constructor** of `std::thread` creates the thread with **no thread function**
- then, that thread exits right after its initiation
    * because the thread treats its lack of thread function as the **immediate returning**
- it's worth noting that you **cannot** specify an **initial function** for a `terminated` thread
    * unless you **transfer the ownership** which is covered later in this post
- therefore, it's highly recommended to **specify** the **thread function** to the **constructor** of thread object

### Passing Temporary Functor
If you pass a **temporary function object**, the compiler might interpret it as a **function declaration**, not a thread construction.
```cpp
std::thread myThread(FunctionObject());     // Won't start (interpreted as function declaration)

// Correct ways to start the thread:
std::thread myThread((FunctionObject()));   
std::thread myThread{FunctionObject()};
```
- In order to avoid this, use **additional parentheses** or **brace (uniform) initialization**:
- It's worth noting that although `lambda` expressions also return **temporary unnamed function objects**, but they **can** be used directly without the issue mentioned above: 
    ```cpp
    std::thread myThread([] { /* do something */ });  // Starts normally
    ```

### Managing the Lifetime of std::thread
After initiating a thread, you must **explicitly decide** `how to manage` it **before** the `std::thread` object is **destroyed**.
- it's worth noting that the **decision** is tied to the **lifetime** of the `std::thread` object, **not** the **lifetime** of the `initial functor`.
    * This means you **can** make the decision **after** the `thread function` **returns**, as long as the std::thread object itself still exists.

### `.join()` OR `.detach()`
Threr are **two options** for your decision

**1. `.join()`**
- this call **waits** for the thread to **complete execution**. 
- After calling `.join()`, you **cannot** interact with the thread object since the **thread** is `terminated`.

**2. `.detach()`**
- this call **separates** the thread from the std::thread object, allowing it to **run independently**. 
- `Detached` threads are often referred to as `*daemon threads*`
    * and they are typically **long-running**.
- It's worth noting that you need to ensure that any **data** accessed by a `detached` thread remains **valid**.
   * because accessing `destroyed` objects results in **undefined behavior**.

### No Decision
If you forget to call `.join()` or `.detach()` before the destruction of the std::thread object
- Then the **destructor** calls `std::terminate`.


## Exception Handling with `std::thread`
When managing threads, the handling of `.join()` and `.detach()` requires careful consideration:
- **`.detach()`**:  
    * The specific place where you call `.detach()` is typically **less critical** compared to `.join()`.
    * It is often called **immediately after** the thread is **launched**.
- **`.join()`**:  
    * Deciding the **proper placement** of `.join()` is **crucial**.
    * If an **exception** occurs before `.join()` is called, the call may be **skipped**, leading to **potential issues**.
        * Therefore, you need to provide special care for the threads of `.join()`

### Using RAII for Exception Safety
To ensure that `.join()` is always called, even if an exception occurs, you can use the `RAII` (Resource Acquisition Is Initialization) idiom.
```cpp
// class definition
class ThreadGuard {
public:
    explicit ThreadGuard(std::thread &inputT) : t(inputT) {}

    ~ThreadGuard() {
        if (t.joinable())
            t.join();       // Automatically join the thread on destruction
    }

    // Prevent copying to avoid multiple `ThreadGuard` objects managing the same thread
    ThreadGuard(const ThreadGuard&) = delete;
    ThreadGuard& operator=(const ThreadGuard&) = delete;

private:
    std::thread& t;
};
```
```cpp
// example use
void threadFunction() {
    // Some work
}

void example() {
    std::thread t(threadFunction);
    ThreadGuard guard(t);  // Ensures `.join()` is called, even if exceptions occur

    // Perform other operations...
    // No need to manually call `.join()`, as `ThreadGuard` handles it
}
```
- The **destructor** of `ThreadGuard` is invoked **earlier** than the **destructor** of the `std::thread` object it manages
    * because the `ThreadGuard` is constructed **after** the `std::thread` object.
- Hence, if you call `.join()` with joinable check inside the **destructor** of `ThreadGuard` object, you can **ensures** that `.join()` is **called**
    * when the `ThreadGuard` object goes out of scope, preventing resource leaks or runtime errors.
- It's worth noting that calling `.join()` on a **non-joinable thread**, such as `terminated` thread, results in **undefined behavior**, typically a **runtime error**.
    * Therefore, it's highly **recommended** to call `.joinable()` in order to prevent these issues by checking if the thread is joinable **before** attempting to join it.


## Argument Passing to `std::thread`
You can pass **arguments** to the **thread function** by providing them after the callable in the constructor of `std::thread`. 

### Behavior of Argument Passing
The process of argument passing happens in **two stages**:
1. From the **original code** to the **constructor** of `std::thread`.
    * This phase is slightly **odd**
2. From the **constructor** to the **thread function**.
    * This phase **works** as what you **anticipated**

### Rvalue Arguments
When passing `rvalues`, everything **works as expected** due to `perfect forwarding` support in the constructor of `std::thread`.

### Lvalue Arguments
For `lvalue` arguments, there is a need to examine it more thoroughly

**1. `Copy` Behavior**
```cpp
struct MyClass {
    MyClass() { std::cout << "Constructor called" << std::endl; }
    MyClass(const MyClass &a) { std::cout << "Copy constructor called" << std::endl; }
};

void f(MyClass a) {
    std::cout << "hello" << std::endl;
}

int main() {
    MyClass a;
    std::thread myThread(f, a);
    ThreadGuard g(myThread);

    return 0;
}
/*
Output:
Constructor called
Copy constructor called
Copy constructor called
hello
*/
```
- If you pass the `copy` of lvalue, the the result is what you **anticipated** as passing `rvalues`
- However, it's worth noting that `copy` happens **twice** because there are 2 paasing processes to occur
- If you want to **avoid** this situation, you may call `std:::ref()` utility function for the first stage
    ```cpp
    // same code above

    int main() {
        MyClass a;
        std::thread myThread(f, std::ref(a));
        ThreadGuard g(myThread); 

        return 0;
    }
    /*
    Output:
    Constructor called
    Copy constructor called
    hello
    */
    ```
    * It's quite important to keep in mind that the `lvalue` parameters are **not** passed as `references` to the **constructor** of `std::thread` object just because the **thread function** takes them as `references`
        + By **default**, they are passed as `copy` to the **constructor** (first phase)
        + They are passed as `references` to the **constructor** of the `std::thread` object **only if** they are passed with `std::ref()`
        + This works in case where the **thread function** takes them as `copy`
   
**2. `Reference` Behavior**

The code below shows the example related to passing `references`
```cpp
// Without std::ref
struct MyClass {
    MyClass() { std::cout << "Constructor called" << std::endl; }
    MyClass(const MyClass &a) { std::cout << "Copy constructor called" << std::endl; }
};

void f(const MyClass &a) {
    std::cout << "hello" << std::endl;
}

int main() {
    MyClass a;
    std::thread myThread(f, a);
    ThreadGuard g(myThread);

    return 0;
}
/*
Output:
Constructor called
Copy constructor called
hello
*/


// Using std::ref
struct MyClass {
    MyClass() { std::cout << "Constructor called" << std::endl; }
    MyClass(const MyClass &a) { std::cout << "Copy constructor called" << std::endl; }
};

void f(MyClass &a) {
    std::cout << "hello" << std::endl;
}

int main() {
    MyClass a;
    std::thread myThread(f, std::ref(a)); // Pass lvalue reference using std::ref
    ThreadGuard g(myThread); // Automatically joins thread

    return 0;
}

/*
Output:
Constructor called
hello
*/
```

### Detached Threads and Local Objects
When using `.detach()` with a local object, be cautious about argument passing.
- If the **object passed** to the thread function is `destroyed` while the `detached` thread is **still running**
   * there might be a chance where it attempts to **access** a `destroyed` object, resulting in **undefined behavior**.
- this applied to **passing handles** to local object as well
   * which means that passing argument as `copy` can be **problematic** if it is the **handle** to `local object`


## Transferring Ownership
The `std::move()` function is used to transfer ownership of a thread between `std::thread` objects. 

### Basic Transferring
```cpp
std::thread t1(f);                // `t1` owns the thread running `f`
std::thread t2 = std::move(t1);   // Ownership is transferred to `t2`
```
- After the transfer, the **original** `std::thread` object (`t1`) **no longer manages** a thread.
    * which means that it does **not** need to call `.detach()` or `.join()` (and it **cannot**).

### Transferring Ownership Back
It is **possible** for a **moved-from** `std::thread` object to **regain** ownership from another `std::thread` object:
```cpp
std::thread t1(f);
std::thread t2 = std::move(t1);   // Transfer ownership to `t2`
t1 = std::move(t2);               // Transfer ownership back to `t1`
```

### Terminated std::thread and Ownership Transfer
A **terminated** `std::thread` object **can** also acquire ownership from a running thread:
```cpp
std::thread t1(f);                // `t1` owns the thread running `f`

std::thread t2;                   // Default-constructed `std::thread` (no thread)
// or
std::thread t2(f);
t2.join();                        // Wait until t2 finishes executing
//or
std::thrad t2(f);
t2.detach();                      // Detach on a separate thread

t2 = std::move(t1);               // `t2` now owns the thread, `t1` is empty
```
- It's worth noting that the `std::thread` object is `terminated` when its
    * **default constructor** is called
    * `.join()` is called
    * `.detach()` is called
- For `.detach()`, there might be a situation where the **exising thread function** is **not** executed 
- You can resolve this issue by using
    * `.join()` instead
    * **Synchronization** which would be covered in the later chapter

### Transferring to an Active std::thread
```cpp
std::thread t1(f1);
std::thread t2(f2);
t2 = std::move(t1);               // ERROR: `t2` already manages a thread
```
- It's worth noting that if you attempt to `move` a thread into an `std::thread` object that manages a **thread** which is still **active** (not `terminated`, not `moved-from`), the program will call `std::terminate()`.

### Upgraded ThreadGuard
The `ThreadGuard` class which we implemented above now can be **upgraded** so  that ownership is properly managed.
```cpp
class ThreadGuard {
public:
    // Allow rvalue only, then transfer ownership to ThreadGuard's data member
    explicit ThreadGuard(std::thread&& inputT) : t(std::move(inputT)) {}

    ~ThreadGuard() {
        if (t.joinable())
            t.join();
    }

    // Allow manual joining if needed
    void join() {
        if (t.joinable())
            t.join();
    }

    // Prevent copying to avoid multiple `ThreadGuard` objects managing the same thread
    ThreadGuard(const ThreadGuard&) = delete;
    ThreadGuard& operator=(const ThreadGuard&) = delete;

private:
    std::thread t;
};
```
- With this update, the **exisiting** `std::thread` object is converted into `changed-from` states when you apply `ThreadGuard` to it
    * so that **only** `ThreadGuard` object can **manage** the thread
- Example Usage
    ```cpp
    void f() {
        std::cout << "Thread running" << std::endl;
    }

    int main() {
        std::thread t(f);
        ThreadGuard guard(std::move(t)); // Transfers ownership to `ThreadGuard`
        // now you can ignore t

        // No need to manually join; `ThreadGuard` handles cleanup.
        return 0;
    }
    ```
- With the upgraded ThreadGuard, thread management becomes safer and more robust.


## Extra Details

### `std::thread::hardware_concurrency()`
`std::thread::hardware_concurrency()` returns `the number of threads` that can truly run concurrently on the **current hardware** for a given program execution. 
- This can help determine the optimal number of threads to use.
    ```cpp
    #include <thread>
    #include <vector>
    #include <algorithm>
    #include <iostream>

    constexpr int MY_EXPECTATION_MAX = 24;

    void f(int id) {
        std::cout << "Thread " << id << " running\n";
    }

    int main() {
        int realMaximumNumber = std::min(std::thread::hardware_concurrency(), MY_EXPECTATION_MAX);

        realMaximumNumber--; // Reserve one thread for the main thread
        std::vector<std::thread> threads(realMaximumNumber);

        // Launch threads
        for (int i = 0; i < realMaximumNumber; i++)
            threads[i] = std::thread(f, i);
        // Join threads
        for (auto& thread : threads)
            thread.join();

        return 0;
    }
    ```
- It's worth noting that if you create **more threads** than the **hardware can handle**, `performance` can **decrease** due to context switching overhead.
   * This phenomenon is called `oversubscription`.

### get_id()
Each **thread** has a `unique ID` which can be obtained by calling `get_id()` 
There are **two ways** to obtain **Thread IDs**

1. From a `std::thread` **object**:
    * Use the `.get_id()` method
        ```cpp
        std::thread t([] { /* do something */ });
        std::cout << "Thread ID: " << t.get_id() << std::endl;
        t.join();
        ```

2. From the **Current Thread**
    * Use `std::this_thread::get_id()` to obtain the ID of the thread currently executing
        ```cpp
        #include <iostream>
        #include <thread>

        void f() {
            std::cout << "Current Thread ID: " << std::this_thread::get_id() << std::endl;
        }

        int main() {
            std::thread t(f);
            t.join();
            return 0;
        }
        ```
- If a `std::thread` object does **not** manage an **active** thread, calling `.get_id()` returns a **default-constructed** `std::thread::id` object, representing `no thread`.
- **Thread IDs** are useful for managing `thread-specific` data or debugging.
- **Thread IDs** can be used as `keys` in ordered (`std::map`) or unordered (`std::unordered_map`) **associative containers**:
    ```cpp
    #include <unordered_map>
    #include <thread>
    #include <iostream>

    std::unordered_map<std::thread::id, std::string> threadNames;

    void f(const std::string& name) {
        threadNames[std::this_thread::get_id()] = name;
        std::cout << "Thread Name: " << name << ", ID: " << std::this_thread::get_id() << std::endl;
    }

    int main() {
        std::thread t1(f, "Worker 1");
        std::thread t2(f, "Worker 2");

        t1.join();
        t2.join();

        return 0;
    }
    ```

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}