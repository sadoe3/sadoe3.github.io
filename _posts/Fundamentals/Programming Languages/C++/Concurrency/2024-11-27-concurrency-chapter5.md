---
title: "C++ Concurrency : Low-Level Synchronization"

categories:
    - cpp

tags:
    - [C++, Programming Language, Concurrency, low-level synchronization, synchronization, atomic, memory ordering]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-27
---

# C++ Concurrency : Chapter 5

> 이 포스트는 `C++ Concurrency in Action (2nd ed.)`을 바탕으로 작성되었습니다.


## Atomic Types
Every object in a C++ program has a **modification order**, which consists of all the `writes` to that object across all threads, starting from its initialization
- In most cases, this order will **vary** between runs
    * but during a single run, all threads in the system must **agree** on the **modification order**
- If the object is of an `atomic` type
    * the **compiler** ensures that the necessary **synchronization** is in place
- Otherwise, **you**, as the programmer, are responsible for ensuring that sufficient **synchronization** exists to make sure all threads agree on the modification order of each object, using the synchronization techniques we've discussed so far

### Atomic Operations
An **`atomic` operation** is an **indivisible** operation, which means it is either **fully** executed or **not** executed at all
- For instance, when the `load()` that reads the value of an `atomic` object is performed, the following is guaranteed
    * All modifications to that object are also `atomic`
    * The `load()` will either retrieve the **initial** value of the object or the **most recent** value stored by one of the modifications
- In C++, to perform an `atomic` operation, you must use an **atomic type** in most cases

### Lock-Free Operations
It's worth noting that `atomic` operations are typically **`lock-free`**
- `Lock-free atomic` operations allow **concurrent** threads to safely perform operations on shared data **without** blocking each other using **traditional** locking mechanisms
- The key use case for `atomic` operations is as a **replacement** for operations that would otherwise require a `std::mutex`-like object for **synchronization** 
- Although it is **possible** for `atomic` operations to internally use a `std::mutex`, this is **not** recommended due to potential **performance** issues

### The Standard Atomic Types
There are **two main types** of standard `atomic` types used for atomic operations
- **Instances of the `std::atomic<>` class template**:
    * For **built-in** types such as `bool`, `int`, and pointers, `atomic` operations are **typically** `lock-free` on modern hardware
        + but this is **not guaranteed** for all platforms or types
    * Hence it's recommended to **check** whether `atomic` operations are `lock-free` for specific types on the current platform using the `ATOMIC_<TYPE>_LOCK_FREE` macros
        + e.g., `ATOMIC_BOOL_LOCK_FREE()`
        + and there are 3 possible outcomes
        + `2` means `atomic` operations are **always** `lock-free`
        + `1` means `atomic` operations are **supported** but **not guaranteed** to be `lock-free`
            - The implementation may use locks or other mechanisms internally.
        + `0` means `atomic` operations are **never** `lock-free`
    * For **other types**, the C++ standard does **not guarantee** `lock-free` behavior
        + You can check if an instance is `lock-free` by calling `.is_lock_free()`
- **`std::atomic_flag`**
    * This is a simple `bool` type that **does not provide** an `.is_lock_free()` method, as it is **always** `lock-free`
    * **Operations** on this type are required to be `lock-free`

### Standard Atomic Types in C++
All standard `atomic` types are defined in the `<atomic>` header file
- **All operations** on these types are `atomic`
- Starting from C++17, all `atomic` types have a `static constexpr` data member `::is_always_lock_free()`
    * which is `true` if and only if the `atomic` type is `lock-free` on **all** supported hardware **platforms**

### Special Properties of Standard Atomic Types
The standard `atomic` types are **not** `copyable` or `assignable` in the conventional sense
- They have **no** `copy constructors` or `copy assignment operators`
- However, they **do** support `assignment` from and `implicit conversion` to the corresponding **built-in** types
- They also **support** `compound assignment operators` and `increment/decrement operators` where applicable

### Custom Atomic Types
Although most low-level synchronization can be achieved with the two standard `atomic` types (`std::atomic<>` and `std::atomic_flag`)
- It is also **possible** to implement your own **custom atomic class**


## Atomic Operations
It's important to note that while threads must **agree** on the modification orders for **individual** objects, they do **not** need to agree on the `relative order` of operations across **separate** objects
- This implies that you need to **handle** this `relative ordering` explicitly

### Operations on `std::atomic_flag`
`std::atomic_flag` provides **limited** features because its core purpose is to serve as a `building block` for more complex synchronization mechanisms
- It is **not** designed to be used on **its own** except under special circumstances, however, it's worth knowing how it works 
- A `std::atomic_flag` can be in one of two `states`
    * `set` or `clear`
- It should be **initialized** with `ATOMIC_FLAG_INIT`
    * which initializes the flag to a `clear` state
- After initialization, you can perform the following operations:
    * **Destroy** it
        + by calling its `destructor`
    * **Clear** it 
        + by calling `.clear()`
    * **Set** it and **query** the previous value 
        + by calling `test_and_set()`

### Memory Ordering Overview
In **`atomic` operations**, **`memory ordering`** determines how operations in different threads are **synchronized** and observed in terms of their execution order
- Here's an overview of the commonly used `memory orders`
- **`std::memory_order_relaxed`**
    * **No** ordering guarantees
    * Operations can be **re-ordered** for performance but should be used with caution
- **`std::memory_order_consume`**
    * Similar to `acquire`, but applies only to **data-dependent** operations
- **`std::memory_order_acquire`**
    * **Guarantees** that memory **operations** before the `atomic` operation in the program **happen before** the `atomic` operation
    * This is typically used when loading data from a shared resource after acquiring a `lock` or `flag`
    * Often used with `test_and_set()` for `locking`
- **`std::memory_order_release`**
    * **Guarantees** that memory **operations** after the `atomic` operation **happen after** the `atomic` operation
    * Typically used with `clear()` for `unlocking`
- **`std::memory_order_acq_rel`**
    * **Combines** both `acquire` and `release` semantics, ensuring synchronization before and after the `atomic` operation
- **`std::memory_order_seq_cst`**
    * **Sequential consistency**, the strictest ordering, where **all operations** appear to happen in the `same order` across `all threads`
- If you do not specify a memory order, the **default** is `std::memory_order_seq_cst`
- More detailed information will be covered later in this chapter
    * Hence, don't worry if you don't understand what this ordering means right now

### Example: Using `std::atomic_flag`
```cpp
#include <iostream>
#include <thread>
#include <atomic>

std::atomic_flag flag = ATOMIC_FLAG_INIT; // Initialize the atomic flag

// Function simulating a task using an atomic flag for synchronization
void task(int id) {
    // Spinlock: wait for the flag to be clear (i.e., the flag is not set)
    while (flag.test_and_set(std::memory_order_acquire))
        ;// Busy-wait (spinlock)

    // Once we acquire the flag, do the task
    std::cout << "Thread " << id << " is working...\n";
    std::this_thread::sleep_for(std::chrono::seconds(1));

    // Set the flag back to clear (release the lock)
    flag.clear(std::memory_order_release);
}

int main() {
    std::thread t1(task, 1);
    std::thread t2(task, 2);
    std::thread t3(task, 3);

    t1.join();
    t2.join();
    t3.join();
    return 0;
}
```
- In this example, you can think of `test_and_set()` as **conceptually similar** to `lock()` and `clear()` as being like `unlock()`
    * Like `lock()`, `test_and_set()` is used to **acquire** the `lock`
        + It ensures that only **one thread** can **acquire** the `flag` at a time
        + If the `flag` is **already set**, it indicates that **another thread** has **locked** it
    * `clear()` atomically clears the flag, **releasing** the `lock`
        + Once the `flag` is **cleared**, **other threads** can **acquire** the `flag` by calling `test_and_set()`
- However, there are some important **distinctions** in behavior and purpose
    * Unlike **traditional** mechanisms, `test_and_set()` **doesn't block** the thread in the conventional way
        + Instead, it uses **busy-waiting** (`spinlock`), where the thread **continually checks** the flag until it can acquire it
    * In a `std::mutex`, the thread is put to **sleep** until the lock becomes available

### Operations on `std::atomic<bool>`
`std::atomic<bool>` supports more full-featured **`atomic` operations** than `std::atomic_flag`, but it still **cannot** be `copied` or `assigned` to from another `std::atomic<bool>`
```cpp
std::atomic<bool> a(true);
a = false;
```
- However, it **can** be `constructed` from a **non-`atomic`** `bool`, so it can initially be set to `true` or `false`
    * and you can **assign** values from a **non-`atomic`** `bool`
- It's worth noting that the `assignment operator` for `std::atomic<bool>` **differs** from the normal assignment operator in that
    * it returns the **value** of the `right-hand side` operand (old one) rather than a **reference** to the `left-hand side` (new one)
- For `std::atomic<bool>`, you perform **writes** using
    * `store()` to set it to `true` or `false`, or
    * `exchange()` to replace the stored value atomically while retrieving the previous value (a read-modify-write operation)
- **Reads** can be performed with
    * `load()` or by using `implicit conversion` to a **non-`atomic`** `bool`
- The code below shows the example of this
    ```cpp
    std::atomic<bool> b;
    // Read
    bool x = b.load();
    x = b;
    // Write
    b.store(true);
    s = b.exchange(false);
    ```

### `compare_exchange()` 
`compare_exchange()` is crucial in **`atomic` operations**
```cpp
bool expected = true;
std::atomic<bool> b = false;
while (b.compare_exchange_weak(expected, false) == false && expected);      // necessary loop for weak
```
- It **compares** the value of an `atomic` object with a supplied `expected value`
- If they are **equal**
    * It **stores** the `desired value`
    * Then it returns `true`
- If they are **not equal**
    * The `expected value` is **updated** to the value of the `atomic` object
    * Then it returns `false`
- There are **two types** of these methods
    * `compare_exchange_weak()`
        + For this type, a *`spurious failure`* may occur
            - where it returns `false` even though the values were **equal**
        + This is most likely on machines **without** a native `compare-and-exchange` instruction
        + If the processor **cannot guarantee** atomicity (e.g., due to thread switching), this could **happen**
        + Thus, you should use it in a **loop**
    * `compare_exchange_strong()`
        + If you don't want this loop, use `compare_exchange_strong()`
        + This one **guarantees** that it will **only** return `false` if the values are **not equal**
    * **Guideline**
        + If **calculating** the value to be stored is **time-consuming**, use `compare_exchange_strong()`
        + Otherwise, use `compare_exchange_weak()` for performance reasons

### Operations on `std::atomic<T*>`
**Similar** to `std::atomic<bool>`, `std::atomic<T*>` supports **`atomic` operations** such as `store()`, `exchange()`, and `compare_exchange()`, along with **pointer arithmetic** operations like `pre/post-increment` and `pre/post-decrement`
- Operations also include `fetch_add()` and `fetch_sub()` for **atomic** additions and subtractions
    * These operations can specify memory ordering as needed.

### Operations on Integral Types
**All operations** available for `std::atomic<T*>` are also **available** for `integral types`
- However, `division`, `multiplication`, and `shift` operations are **not** supported
- This limitation is typically **not significant**
    + as **`atomic` operations** with `integral types` are often used for `counters`, where `increment` and `decrement` operations are **sufficient**

### Operations on User-Defined Types
You can use a **user-defined** type as a parameter of `std::atomic<>`
- But the type must meet certain **conditions**
    * It must have a **trivial** `copy-assignment` operator and
        + meaning that **no** `virtual functions` or `virtual base classes`
    * Every `base class` and `non-static data member` of the type must also have a **trivial** `copy-assignment` operator
- This ensures the **compiler** can **perform** `atomic assignment operations` **without** `user-written code`
- While you **cannot** perform arithmetic operations like `.fetch_add()` on user-defined types
    + you **can** use other atomic methods (like `load()` and `store()`)
- It's important to note that `compare_exchange()` on user-defined types performs a **bitwise comparison** (similar to `memcmp()`)
    + meaning that any **user-defined** comparison operations will **not** be used

### Free Functions for Atomic Operations
There are also **non-member functions** that perform **`atomic` operations** on various `atomic` types
```cpp
std::atomic_store(&atomicObject, newValue);
std::atomic_store_explicit(&atomicObject, newValue, std::memory_order_release);

// For atomic_flag
std::atomic_flag_clear(&atomicFlag);
```
- These functions take a **pointer** to the `atomic` object as their `first parameter` due to their **C compatibility**
- Although `std::shared_ptr<T>` is **not** `atomic` by default, the C++ Standard Library **provides** functions to use them `atomically`
    ```cpp
    std::shared_ptr<int> a = std::make_shared<int>(3);
    std::shared_ptr<int> local = std::atomic_load(&a);
    ```
- However, you must **ensure** that there are **no concurrency issues** with this usage, as it does **not guarantee** `atomicity`


## Memory Ordering
- e are **6** `memory ordering` options for `atomic` types:
    * `memory_order_relaxed`
    * `memory_order_consume`
    * `memory_order_acquire`
    * `memory_order_release`
    * `memory_order_acq_rel`
    * `memory_order_seq_cst`
- By **default**, the `memory order` is `memory_order_seq_cst`.
- These 6 options correspond to 3 `main models`:
    * **Sequentially Consistent** 
        + `memory_order_seq_cst`
    * **Relaxed** 
        + `memory_order_relaxed`
    * **Acquire-Release**
        + `memory_order_consume`, `memory_order_acquire`, `memory_order_release`, `memory_order_acq_rel`
- Different `memory models` can have **varying** performance costs depending on the CPU architecture
    * Therefore, choosing the `right model` based on the specific situation can help improve performance

### Sequentially Consistent Ordering
In `sequentially consistent` ordering, **all operations** on `different threads` **always** appear in the `same order`, as if they were performed on a `single thread`
- If a `sequentially consistent` **store** is used, then a `sequentially consistent` **load** must also be used to maintain synchronization
- `Sequential consistency` is **intuitive** but **expensive** because it forces **global synchronization** across `all threads`, ensuring that `all threads` see the `same order` of operations
    * This could involve **additional synchronization** between processors which decreases the performance
- Therefore, if **performance** is a concern, consult the **documentation** for your target processor architecture before using `sequentially consistent` ordering, as it may affect performance

### Non-Sequentially Consistent Ordering
In **non-`sequentially consistent`** ordering, `threads` do **not have to** agree on the **order** of operations
- It allows `threads` to have **different orders** of the `same operations`, making it more **flexible** than `sequentially consistent` ordering
    * This can be useful when you do **not** need **strict synchronization** between threads

### Relaxed Ordering
In `relaxed` ordering, the compiler or hardware can `reorder` operations in a way that might affect how other threads perceive the sequence of events
- **`Same thread`**
    * Operations are ordered normally
    * They are executed as you **expected**
- **`Different threads`**
    * Operations can be `reordered`, which might lead to **unexpected** results
- The code below shows the example of `relaxed` ordering
    ```cpp
    std::atomic<bool> x(false), y(false);
    std::atomic<int> z(0);

    void writeXY() {
        x.store(true, std::memory_order_relaxed);   
        y.store(true, std::memory_order_relaxed);  
    }

    void readYX() {
        while (!y.load(std::memory_order_relaxed));   
        if (x.load(std::memory_order_relaxed))     
            ++z;
    }

    int main() {
        x = false;
        y = false;
        z = 0;
        std::thread a(writeXY);
        std::thread b(readYX);
        a.join();
        b.join();
        assert(z.load() != 0);  
    }
    ```
- In this example, `assert()` **could** be fired because there's **no guarantee** that `x.store()` **happens before** `y.store()` from the perspective of the `readYX thread`
    * The compiler or hardware may `reorder` the operations, leading to a possible `data race`
- Hence it's recommended to **avoid** using `relaxed` memory ordering unless absolutely necessary, and use it with extreme caution to prevent `data races` and **unexpected** results

### Acquire-Release Ordering
`Acquire-release model` ensures that operations on `one thread` (the `store`) **synchronize with** operations on `another thread` (the `load`), **without** needing `global synchronization`
- In this model
    * **`Atomic` loads** are `acquire` operations (`memory_order_acquire`)
    * **`Atomic` stores** are `release` operations (`memory_order_release`)
    * **`Atomic` read-modify-write** operations can be either `acquire`, `release`, or both (`memory_order_acq_rel`)
- The code below shows the example of this
    ```cpp
    std::atomic<bool> x(false), y(false);
    std::atomic<int> z(0);
    void writeXY() {
        x.store(true, std::memory_order_relaxed);   
        y.store(true, std::memory_order_release);   
    }
    void readYX() {
        while (!y.load(std::memory_order_acquire)); 
        if (x.load(std::memory_order_relaxed))
            ++z;
    }

    int main() {
        x = false;
        y = false;
        z = 0;
        std::thread a(writeXY);
        std::thread b(readYX);
        a.join();
        b.join();
        assert(z.load() != 0);    
    }
    ```
- In this case, `assert()` **cannot** be fired because `y.store()` **synchronizes with** `y.load()` via `acquire-release` ordering
    * This means `y.store()` **happens before** `y.load()` thanks to `acquire-release`, and `x.store`() **happens before** `y.store()` because they are executed on the `same thread`  
    * Hence `x.store()` **happens before** `y.load()`
    * Then, `x.load()` will always return `true` if `y.load()` is `true`

### Transitive Ordering for Acquire-Release Ordering
`Acquire-release` ordering is **transitive**
- If `thread A` synchronizes with `thread B`, and `thread B` synchronizes with `thread C`, then `thread A` synchronizes with `thread C`
- The code below shows the example of this
    ```cpp
    std::atomic<bool> sync1(false), sync2(false);
    std::atomic<int> data[2];

    void doThreadA() {
        data[0].store(43, std::memory_order_relaxed);
        data[1].store(50, std::memory_order_relaxed);
        sync1.store(true, std::memory_order_release);
    }

    void doThreadB() {
        while (!sync1.load(std::memory_order_acquire));
        sync2.store(true, std::memory_order_release);
    }

    void doThreadC() {
        while (!sync2.load(std::memory_order_acquire));
        assert(data[0].load(std::memory_order_relaxed) == 43);
        assert(data[1].load(std::memory_order_relaxed) == 50);
    }

    int main() {
        std::thread t1(doThreadA);
        std::thread t2(doThreadB);
        std::thread t3(doThreadC);
        t1.join();
        t2.join();
        t3.join();
    }
    ```
- In this case, `t1` happens before `t3` because `sync1` synchronizes between `t1` and `t2`, and `sync2` synchronizes between `t2` and `t3`

### Consume Ordering
`memory_order_consume` is specifically for **data dependencies**
- This ordering allows loads to be `reordered`, but it ensures that dependent operations are sequenced correctly
- It's worth noting that C++17 recommends **not** to use `memory_order_consume`, as compilers often convert it to `memory_order_acquire` for practical reasons

### Fences
`Fences` are **global operations** that enforce `memory ordering` constraints **without** modifying any data
```cpp
std::atomic<bool> x(false), y(false);

void write_x_then_y() {
    x.store(true, std::memory_order_relaxed);          
    std::atomic_thread_fence(std::memory_order_release);    
    y.store(true, std::memory_order_relaxed);          
}

void read_y_then_x() {
    while (!y.load(std::memory_order_relaxed));     
    std::atomic_thread_fence(std::memory_order_acquire);    
    if (x.load(std::memory_order_relaxed))     
        ++z;
}

int main() {
    x = false;
    y = false;
    z = 0;
    std::thread a(write_x_then_y);
    std::thread b(read_y_then_x);
    a.join();
    b.join();
    assert(z.load() != 0);   
}
```
- They are commonly used with `memory_order_relaxed` to ensure proper synchronization
- The fence ensures that code **before** `std::atomic_thread_fence(std::memory_order_release)` in `one thread` always **happens before** the code **after** `std::atomic_thread_fence(std::memory_order_acquire)` in `another thread`

### Non-Atomic
The another **benefit** of using **`atomic` operations** is that they can enforce an ordering on **`non-atomic` operations**, preventing the undefined behavior caused by data races
```cpp
bool x = false;

void write_x_then_y() {
    x = true; 
    std::atomic_thread_fence(std::memory_order_release);    
    y.store(true, std::memory_order_relaxed);          
}

void read_y_then_x() {
    while (!y.load(std::memory_order_relaxed));     
    std::atomic_thread_fence(std::memory_order_acquire);    
    if (x)     
        ++z;
}

int main() {
    x = false;
    y = false;
    z = 0;
    std::thread a(write_x_then_y);
    std::thread b(read_y_then_x);
    a.join();
    b.join();
    assert(z.load() != 0);   
}
```
- In this case, `fences` ensure that there is **no** `data race` for `x`, even though `x` is **not** `atomic`


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}