---
title: "C++ Concurrency : Synchronization"

categories:
    - cpp

tags:
    - [C++, Programming Language, Concurrency, synchronization, condition variable, future, latch, barrier]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-25
---

# C++ Concurrency : Chapter 4

> 이 포스트는 `C++ Concurrency in Action (2nd ed.)`을 바탕으로 작성되었습니다.

## `std::condition_variable`

### Why `std::condition_variable`?
Suppose that `Thread A` needs to **wait** for `Thread B` to complete its **operation**
- There are 3 ways to handle this with **shared flag**
- **Quick and Dirty Approach**
    ```cpp
    while(true) {
        m.lock();
        if(flag) {
            m.unlock();
            break;
        }
        m.unlock();
    }
    ```
    * Although this code might work as you expected,
    * Constantly checking the flag **wastes** `processing time`
    * The flag **cannot** be **locked** by `another thread` while it's being checked, and it's almost **always locked** because this code keeps iterating
- **Improved Approach from First Option**
    ```cpp
    m.lock();
    while(!flag) {
        std::this_thread::sleep_for(std::chrono::milliseconds(100));    // sleep for 100ms
        m.unlock();
    }
    ```
    * Because this code **sleeps** for certain amount of time, we can reduce wasteful processing time
    * However, still the `appropriate sleep period` is **hard to determine**
- **`std::condition_variable`**
    * `Thread A` **waits** for a condition to be satisfied
    * `Thread B` **notifies** `Thread A` (or other `waiting threads`) once its operation is complete, allowing them to **resume processing**
    * Therefore, it's highly recommended to use `std::condition_variable` for efficient and reliable synchronization


### Two Versions
C++ provides **two versions** of condition variables
- **`std::condition_variable`**
    * it's compatible **only** with `std::mutex`.
- **`std::condition_variable_any`**
    * it's compatible with **any** `std::mutex-like` object, including `std::shared_mutex`
- Both are declared in the `<condition_variable>` header
- If you're using **only** `std::mutex`, prefer `std::condition_variable` for better **performance**


### How To Use
The code below shows basic example of how to use `std::condition_variable`
```cpp
#include <condition_variable>
std::condition_variable dataCondition;
std::mutex m;

void threadA() {
	while(isThereMoreData()) {
		const Data data = readData();
		{
			std::lock_guard<std::mutex> guard(m);
			collection.push(data);
		}
		dataCondition.notify_one();
	}
}

void threadB() {
	while (true) {
		std::unique_lock<std::mutex> guard(m);      // std::unique_lock is required
		dataCondition.wait(guard, [] {return !collection.empty(); });

		Data data = collection.front();
		collection.pop();
		guard.unlock();

		processData(data);
		if (isLastData(data))
			break;
	}
}
```
- In order to understand this example, you need to learn how `.wait()` and `.notify_one()` work
    * **Step 1**: `threadB` **locks** the `std::mutex` via `std::unique_lock<std::mutex>`
        + `std::unique_lock` is **required** because `.wait()` need to **lock** and **unlock** the `std::mutex` **directly**
        + which means that, technically speaking, it's **possible** to pass raw `std::mutex` object itself to condition variable, although it's **not recommended**
    * **Step 2**: `threadB` calls `.wait()` on `dataCondition`
        + once `.wait()` is called
        + it calls **`.unlock()`** on the given `std::mutex` object
        + it **blocks** the thread until `dataCondition` is **notified**
    * **Step 3**: `threadA` calls `.notify_one()` or `.notify_all()` on `dataCondition`, which **wakes up** the `waiting thread` (in this case `threadB`)
    * **Step 4**: `threadB` calls `.lock()` on the given `std::mutex` object (in this case `m`)
    * **Step 5**: `threadB` checks the **predicate** (*if provided*)
        + If the **predicate** is `false`, it will **go back** to `waiting`
            * which means it **unlocks** `std::mutex` object and **blocks** again until notified
        + If the **predicate** is `true`, it **proceeds** with its work
        + If **no predicate** is provided, then it **always proceeds** with its work
- If there are `multiple threads` waiting, then call `.notify_all()` instead of `.notify_one()`
    * If you call `.notify_one()` in this case
    * There is **no guarantee** as to `which thread` is **selected**
- It's worth noting that it's highly **recommended** to set the `std::mutex` **data member** as `mutable`
    * This is because there are various circumstances where it should be **locked** or **unlocked** in `const` **member functions**


## Future
In C++, `std::future` allows for asynchronous programming by providing mechanisms to **retrieve a result** from the task **running** in the **background**
- There are two types of futures
    * **`std::future<>`** (unique future)
        + which represents `a single unique instance` that refers to `a specific event`
    * **`std::shared_future<>`** (shared future)
        + which allows `multiple instances` to refer to `the same event`, with all instances becoming ready at the same time
- Both are declared in the `<future>` header file and are **modeled after** `std::unique_ptr` and `std::shared_ptr`
- There is also a **specialization** for `void`, which is used when there is **no associated data** to `return`

### How To Use
The **`std::async`** function template (also declared in `<future>`) is used to start an **asynchronous task**
```cpp
#include <iostream>
#include <future>

int findTheAnswer(int data) {
    std::cout << data << " is passed" << std::endl;
    int answer = 0;
    for (; answer < 100; answer++)
        ;
    return answer;
}

int main() {
    std::future<int> answer = std::async(std::launch::async, findTheAnswer, 3);

    // Do other stuff here...

    // Then get the result
    std::cout << "The answer is " << answer.get() << std::endl;
    return 0;
}
```
- It returns a `std::future` object, where the template type corresponds to the return type of the function you provided to `std::async`
- When you need the value, call `.get()` on the `std::future` object
    * This will **block** the `current thread` until the `result` is `ready`, and then it will **return** the value
    * `result` becomes `ready` when the provided function **returns** a value;

### Regarding `std::async`
- **Deferred Execution:**
    * By **default**, the provided function call is **deferred** until `.get()` is called
    * To ensure the function is **run concurrently**, you need to specify `std::launch::async` as the `first argument`
    * You can **explicitly** set the argument to `std::launch::deferred` if you want **deferred** execution
    * To let the **implementation decide**, you can pass `std::launch::deferred | std::launch::async` as the first argument, allowing it to choose between deferred or asynchronous execution
- `std::async` works **similarly** to `std::thread`
    * **Additional arguments** can be passed after the function, similar to how arguments are passed in `std::thread`
    * If the `first argument` is a pointer to a `member function`, pass the `pointer` or a `reference` (using `std::ref`) to the `object` as the `second argument`
    * `Rvalue references` can be passed in the same way


## `std::packaged_task<>`
A `std::packaged_task<>` is a **class template** for a `callable function` (such as a function, lambda, or functor) that works similarly to `std::async(std::launch::async)`
- You can retrieve the **result** of the function via a `std::future` object
- The difference is that `std::async(std::launch::async)` **calls** the function on separate thread **right after** it's constructed
    * whereas, you can **decide** the **proper timing** to **call** the function which will be run on separate thread by using `std::packaged_task<>`

### Template Parameter
The `std::packaged_task<>` class template is **parameterized** with the `function's signature`
- For instance, `int(const std::string&, double*)` means `int` is the `return type`, and the `function parameters` are `(const std::string&, double*)`
- The type of the parameters is **not strict**
    * which means that `implicit conversions` are **allowed**
    * so if the conversion works, it's perfectly fine

### Basic Example
```cpp
#include <iostream>
#include <future>

int add(int a, int b) {
    return a + b;
}

int main() {
    std::packaged_task<int(int, int)> task(add);
    std::future<int> result = task.get_future();

    // Call operator() directly on the packaged_task (same thread)
    task(2, 3);  // This will execute 'add' on the current thread (main thread)
    
    std::cout << "Result: " << result.get() << std::endl;

    return 0;
}
```
```cpp
// same code above

// Launch the task on a separate thread by calling operator() inside a thread
std::thread t(std::move(task), 2, 3);  // 'task' will execute on the new thread

// Wait for the result from the future
std::cout << "Result: " << result.get() << std::endl;
t.join();
return 0;
```
- When you **construct** a `std::packaged_task`, you should provide a `callable function` (such as a regular function, lambda, or functor) and `its arguments` like `std::thread`
    * At this point, the function does **not execute yet** unlike `std::thread`
    * The function will **only be executed** when the `operator()` of the `std::packaged_task` is called
- You can get the corresponding `std::future` object by calling `get_future()` on a `std::packaged_task`
    * This `std::future` object can be used to **retrieve the result** of the task once it completes
- It's worth noting that calling `operator()` does **not** create a `new thread` by default
    * The `provided function` is executed on the `same thread` that calls `operator()` from the `std::packaged_task<>` object
    * Therefore, if you want the task to **run** on a `different thread`, you need to **move** (since copy is not allowed) the `std::packaged_task<>` object to `that thread` so that `operator()` is called on `that thread` 
- It's quite crucial to notice that calling `operator()` to run the function on separate thread does **not block** any thread
    * **Blocking** happens when you call `.get()` on the `std::future` object but the task **hasn't finished** executing yet
    * it **blocks** until the task finishes and the **result** becomes `ready`

### Advanced Example: GUI Operations
It's often common for `GUI Frameworks` to run **GUI Operations** on the `GUI Thread` **only**
- But you usually want to get the **results** of those operations on `different threads`
- In this case, `std::packaged_task<>` becomes **useful**
- The code below shows the example of this case
    ```cpp
    std::mutex m;
    std::deque<std::packaged_task<void()>> tasks;

    void performGUIOperations() {
        while (isShutdown() == false) {
            getAndProcessGUIMessage();
            std::packaged_task<void()> task;
            {
                std::lock_guard<std::mutex> guard(m);
                // Check for new task
                if (tasks.empty()) 
                    continue;
                task = std::move(tasks.front());
                tasks.pop_front();
            }
            task();  // Execute the task on the GUI thread
        }
    }

    template <typename Func>
    std::future<void> postTaskGUI(Func f) {
        std::packaged_task<void()> task(f);       
        std::future<void> result = task.get_future(); 

        std::lock_guard<std::mutex> gaurd(m);
        tasks.push_back(std::move(task));         // Add task to global task collection
        return result;                             
    }


    // on GUI Thread
    performGUIOperations();
    // on Other Threads
    std::future<void> result = postTaskGUI([] { /* GUI Operation */ });
    result.wait();      // wait until the GUI thread executes the task
    ```
    * A **global collection** of `std::packaged_task<>` objects can be maintained, allowing the `GUI thread` to process tasks in an **infinite loop** while `other threads` interact with it asynchronously


## `std::promise`
- `std::promise` is **similar** to `std::packaged_task` in that
    * both can be used to **run** tasks in `separate threads`, with the `result` being made available in the form of a `std::future`
- However, there are **key differences** between them
    * **`std::packaged_task`** uses a `provided function` to set the result and pass it to a `std::future` object.
    * **`std::promise`** does **not** require a `function`
        + it simply needs to `set a value` that will be passed to the associated `std::future`
- Therefore, the **template parameter** of `std::promise` is the `type of value` that you want to set and pass to the `std::future`

### Basic Example
```cpp
#include <iostream>
#include <thread>
#include <future>

void doWorkerThread(std::promise<int>&& prom) {
    prom.set_value(42);
}

int main() {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();

    std::thread worker(doWorkerThread, std::move(prom));

    std::cout << "Main thread doing some work..." << std::endl;

    // Get the result from the future (this will block until the worker thread sets the value)
    int result = fut.get();  // Blocks here until the worker sets the value

    std::cout << "Result from worker thread: " << result << std::endl;
    worker.join();

    return 0;
}
```
- It's worth noting that **copying** `std::promise` is **not allowed** 
    * If you want to **transfer** a `std::promise` to a `different thread`, you must **move** it
- The `worker thread` **sets the value** to the `std::promise` using the `.set_value()` method.
    * This value is then passed to the associated `std::future` object
- The `main thread` calls `fut.get()` to get the value.
    * This call **blocks** until the **value is set** by the `worker thread` and available through the `std::future` object


## Exception Handling with `std::future`
By default, if an **exception** is thrown in `threadA`, it is **not** possible for `threadB` to **catch** that **exception**
- In such cases, `terminate()` is typically called
- However, when using `std::future`, **exception handling** becomes more **manageable**

### How Exception Handling Works with `std::future`
It's worth noting that when using `std::async` or `std::packaged_task`
- If the **function called by** `async` or `packaged_task` **throws an exception**
    * and **that exception** is **not caught** inside the function
- By default, `std::terminate()` should be called
    * However, it is **not called**
    * Actually, it is **swallowed** in this case
- Instead, **the exception** is **stored** in the corresponding `std::future` object
    * If you call `.get()` on that `std::future` object, **the exception** is **re-thrown**
- Technically speaking, if `.get()` is **not called** on the `std::future` object
    * the program will **continue** executing **without handling the exception**
    * However, this is **not recommended**, as it leaves the exception unhandled

### Example with `std::packaged_task`
```cpp
#include <iostream>
#include <future>
#include <thread>

void doThrowException() {
    throw std::runtime_error("Exception thrown in packaged_task!");
}

int main() {
    // Create packaged_task that wraps a function that throws an exception
    std::packaged_task<int()> task([]() {
        doThrowException();  // Function that throws an exception
        return 42;           // This would not be reached
    });
    std::future<int> fut = task.get_future();
    std::thread t(std::move(task));

    try {
        fut.get();  // This will rethrow the exception thrown in the packaged_task
    } catch (const std::exception& e) {
        std::cout << "Caught exception from packaged_task: " << e.what() << std::endl;
    }

    t.join();
    return 0;
}
```

### Handling Exceptions with `std::promise`
With `std::promise`, you can **explicitly set the exception** to propagate using the `set_exception()` method
```cpp
// For std::promise
try {
    promiseObject.set_value(calculate_value());
} catch(...) {
    // Set a specific exception to propagate
    promiseObject.set_exception(std::current_exception());
    // Alternatively, set a custom exception
    promiseObject.set_exception(std::make_exception_ptr(std::logic_error("foo")));
}
```
- It's worth noting that you can propagate a **custom exception** or the **current exception**

### Destructor Handling of Exceptions
If the `std::future` object **can't retrieve** the value because the `function call operator` for a `std::packaged_task` is **not invoked** and the value is **not set** for a `std::promise` and they are **destroyed**
- The `destructors` of them will store a `std::future_error` exception in the associated `std::future` object 
- When you call `.get()` on that `std::future` object, the `std::future_error` exception is thrown
- The code below shows the example of this
    ```cpp
    std::future<int> result;
    {
        std::packaged_task<int(int, int)> task(add);
        result = task.get_future();
    }

    // Since we didn't call the task, the future will hold a future_error
    try {
        std::cout << "Result: " << result.get() << std::endl;
    } catch (const std::future_error& e) {
        std::cout << "Caught exception: " << e.what() << std::endl;
    }
    ```


## `std::shared_future`
`std::future` is only **movable**, meaning `only one instance` can refer to `a particular result`
- In contrast, `std::shared_future` is **copyable**, allowing `multiple instances` to refer to `the same result`

### Moving from `std::future` to `std::shared_future`
It is possible to **move** a `std::future` into a `std::shared_future`, and you can make any number of copies of the `std::shared_future`
```cpp
std::future<int> f(p.get_future());     

// Move the future to shared_future
std::shared_future<int> sf = std::move(f);  
// You can also implicitly move the future into shared_future
std::shared_future<int> sf(p.get_future());
// Or explicitly use the share method to share the future
std::shared_future<int> sf = f.get_future().share();

// All three options produce the same result
```

### Example
```cpp
#include <iostream>
#include <future>
#include <thread>

void threadA(std::shared_future<int> sf) {
    std::cout << "A: " << sf.get() << std::endl;
}

void threadB(std::shared_future<int> sf) {
    std::cout << "B: " << sf.get() << std::endl;
}

void threadC(std::shared_future<int> sf) {
    std::cout << "C: " << sf.get() << std::endl;
}

int main() {
    auto f = std::async(std::launch::async, [] { return 1; });
    
    std::shared_future<int> sf = std::move(f);
    
    // Pass copies of shared_future object to multiple threads
    std::thread t1(threadA, sf);
    std::thread t2(threadB, sf);
    std::thread t3(threadC, sf);

    t1.join();
    t2.join();
    t3.join();
    return 0;
}
```


## Timeout Handling in C++
By default, a `thread` will **block indefinitely** if it calls `wait()` until it is **notified**
- However, things changed when a `timeout` is applied
- If the `timeout` **expires**, the `thread` **proceeds regardless** of whether the predicate is satisfied or not

### Types of Timeout
There are **two types** of `timeouts` in C++
- **Duration-based timeout**
    * For example, `30 ms`
    * This is specified with the `_for` suffix
- **Absolute timeout**
    * For instance, `18:20:21.0450120320 UTC on December 23, 2013`
    * This is specified with the `_until` suffix

### Recognizing Timeout
Functions that wait with `timeout` value, such as `wait_for()` returns
- `std::future_status` if the object is `std::future`
    * If the `timeout` **occurs** then it returns
        + `std::future_status::timeout`
    * Otherwise, it returns
        + `std::future_status::ready` if the task is done
        + `std::future_status::deferred` if the task is not started yet
- `std::cv_status` if the object is `std::condition_variable`
    * If the `timeout` **occurs** then it returns
        + `std::cv_status::timeout`
    * Otherwise, it returns
        + `std::cv_status::no_timeout` if the task is done

### How Time is Specified in C++
In order to use `timeout`, you need to handle values related to **time**
- If you do **not** use any `C library` or `third-party libraries`
    * Using `std::chrono` **namespace** is the standard way to handle **time-related functionality** in C++
- You need to include the `<chrono>` header to use it

### Key Components in `std::chrono`
`std::chrono` consists of **3 components**
- **Durations**
    * It represents a **span of time**
    * The `std::chrono::duration<>` class template is used to define **time durations** measured in units like seconds, milliseconds, microseconds, etc. 
    * The Standard Library provides a set of **predefined durations**
        + `std::chrono::nanoseconds`, `std::chrono::microseconds`, `std::chrono::milliseconds`, `std::chrono::seconds`, `std::chrono::minutes`, `std::chrono::hours`
    * `C++14` and later makes it easy to represent them
        ```cpp
        using namespace std::chrono_literals;
        auto halfAnHour = 30min;  // 30 minutes
        auto sameThing = std::chrono::minutes(30);  // equivalent to the above
        ```
    * **Conversion** between different **durations** is **implicit** if thereo's **no truncation**
        + For example, converting `minutes` to `seconds` (ex: 60sec) is **fine**, but converting `seconds` to `minutes` (ex: 1.5min) requires **manual handling**
        + This is because **durations** are represented with **integral types**
    * **Explicit conversion** can be done using `std::chrono::duration_cast<>`
        ```cpp
        std::chrono::milliseconds ms(12312);
        std::chrono::seconds s = std::chrono::duration_cast<std::chrono::seconds>(ms);
        ```
    * You can perform **arithmetic operations** on **durations**
        + e.g., adding, subtracting, comparing
- **Clocks**
    * It's **used** to get **time points** (i.e., specific moments in time)
    * `std::chrono::clock` is a class template that defines the **general interface** for `clocks`, but you usually don't instantiate it directly
    * Instead, **specialized clocks** like `std::chrono::system_clock`, `std::chrono::steady_clock`, and `std::chrono::high_resolution_clock` inherit from this template and provide concrete behavior based on the system's clock and the type of timekeeping you need
    * `std::chrono::system_clock`
        + It represents the system's **real-world** (wall-clock) time
        + It's **not steady**, meaning it can **be adjusted**
        + It's typically **synchronized** with **local** system time (unlike `steady_clock`) so that it can be used for **absolute** `timeouts` 
    * `std::chrono::steady_clock`
        + It provides a **monotonic** (increased at a constant rate) clock that increases **steadily**, **unaffected** by system clock changes (like daylight saving time or manual adjustments)
        + It's **not related** to any **absolute calendar time** (such as UTC or **local** time)
        + It's simply a measure of **elapsed time** since the start of the system (or since an arbitrary epoch)
    * `std::chrono::high_resolution_clock`
        + It provides the **highest available precision** (smallest possible tick period), but may alias either `system_clock` or `steady_clock` depending on the platform.
    * It's **possible** to implement your own version of **custom clock** class which has different tick mechanics
- **Time Points**
    * `std::chrono::time_point<>` class template represents **specific points** in time **relative** to a particular **clock**
    * You can get the current time using `std::chrono::system_clock::now()`
    * The **epoch** of the clock is the time when the **clock started ticking**
        + You can retrieve the time **since the epoch** using `::time_since_epoch()`, which returns a `std::chrono::duration`
    * **Arithmetic operations** can be applied to **time points** and **durations**
        ```cpp
        // Typcial use case: offset from some-clock::now()
        std::chrono::high_resolution_clock::now() + std::chrono::nanoseconds(500);  // 500 ns after
        ```

### Using Timeout in C++
There are **3 ways** to use `timeout` in C++
- **Sleep**
    ```cpp
    std::this_thread::sleep_for(std::chrono::milliseconds(100));                                // Sleep for 100 ms
    std::this_thread::sleep_until(std::chrono::system_clock::now() + std::chrono::minutes(1));  // Sleep until 1 minute later
    ```
    * To pause execution for a certain duration, you can use
        + `std::this_thread::sleep_for()` or
        + `std::this_thread::sleep_until()`
    * It's worth noting that `system_clock` should be used for **sleep** functions
    * `steady_clock` is typically used to calcualte the **elapsed time** between certain periods
- **Future or Condition Variable**
    ```cpp
    std::future<int> future = std::async(std::launch::async, [] { return 42; });
    if (future.wait_for(std::chrono::milliseconds(100)) == std::future_status::timeout) 
        std::cout << "Timeout occurred!" << std::endl;
    else
        std::cout << "Result: " << future.get() << std::endl;
    ```
    ```cpp
    std::condition_variable cv;
	std::mutex m;
	std::unique_lock<std::mutex> guard(m);

	std::thread t([&] {std::this_thread::sleep_for(std::chrono::milliseconds(10)); cv.notify_one(); });
	if (cv.wait_for(guard, std::chrono::seconds(1)) == std::cv_status::timeout)
		std::cout << "Timeout occurred!" << std::endl;
	else
		std::cout << "Worked!" << std::endl;
	t.join();
    ```
    * You can apply `timeouts` with `std::future` objects or `std::condition_variable` objects using the `wait_for()` or `wait_until()` functions
- **Mutex with Timed Lock**
    * For `std::mutex-like` objects, there are **specialized** types like `std::timed_mutex`, `std::recursive_timed_mutex`, and `std::shared_timed_mutex` that support **time-based locking** with functions like `try_lock_for()` and `try_lock_until()`
    * For example, you can use `std::shared_timed_mutex` like this
        ```cpp
        std::shared_timed_mutex m;
        std::unique_lock<std::shared_timed_mutex> guard(m);

        // Try to acquire the lock for 1 ms
        if (guard.try_lock_for(std::chrono::milliseconds(1))) {
            // Lock acquired, do something
        } else
            std::cout << "Timeout while trying to acquire the lock." << std::endl;
        ```


## Extra Details

### Parallel QuickSort
Now, you can apply everything we've learned so far to implement concurrent programs
```cpp
std::mutex m;
unsigned numberThreadsInUse = 1;

template<typename T>
std::list<T> quickSortParallel(std::list<T> input) {
	static unsigned MAX_NUMBER_THREADS = std::max<int>(std::thread::hardware_concurrency() - 1, 2);			// Thread Limit
    if (input.empty())
        return input;

    std::list<T> result;
    result.splice(result.begin(), input, input.begin());
    const T& pivot = *result.begin();

    auto dividePoint = std::partition(input.begin(), input.end(), [&](const T& t) {return t < pivot; });
    std::list<T> lowerPart;
	lowerPart.splice(lowerPart.end(), input, input.begin(), dividePoint);
	bool isDeferred = false;
	std::future<std::list<T> > sortedLower;
	{
		std::unique_lock<std::mutex> guard(m);
		if (numberThreadsInUse < MAX_NUMBER_THREADS) {
			sortedLower = std::async(std::launch::async, &quickSortParallel<T>, std::move(lowerPart));
			++numberThreadsInUse;
		}
		else {
			std::cout << "deferred..." << std::endl;
			isDeferred = true;
			sortedLower = std::async(std::launch::deferred, &quickSortParallel<T>, std::move(lowerPart));
		}
	}
    auto sortedHigher = quickSortParallel(std::move(input));

    result.splice(result.end(), sortedHigher);
    result.splice(result.begin(), sortedLower.get());
	if(isDeferred == false) {
		std::unique_lock<std::mutex> guard(m);
		--numberThreadsInUse;
	}

    return result;
}

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
possible output
unsorted
1 42 -23 5 7 40 2 3 6 29 10 17 11 -12 13 -14 15 0 50

deferred...
deferred...
deferred...
deferred...
deferred...

sorted
-23 -14 -12 0 1 2 3 5 6 7 10 11 13 15 17 29 40 42 50
*/
```
- You can adjust the **Thread Limit** according to your needs

### `std::latch`
The `std::latch` provides **blocking** based on the **count** value
```cpp
#include <latch>  

void launchTeam() {
    const unsigned threadCount = (std::thread::hardware_concurrency() - 5); // Arbitrary count
    std::latch done(threadCount);         // Initialize latch with count
    Data data[threadCount];
    std::vector<std::future<void>> threads;
    
    for (unsigned i = 0; i < threadCount; ++i)
        threads.push_back(std::async(std::launch::async, [&done, &data, i] {            // Pass latch as reference
            data[i] = makeData(i);        // Do group work
            done.count_down();            // Decrease latch count
            doMoreStuff();                // Continue other individual work
        }));

    done.wait();  // Wait for latch to reach zero (all threads have finished their work)
    process_data(data, threadCount);  // Process the data after all threads finish
}
```
- It's worth noting that you need to include `<latch>` header file to use `std::latch` **not** `<experimental/latch>`
    * At the time the book was written, <latch> was not part of the standard
        + Therefore, you needed to upgrade your compiler to `c++_latest` and include <experimental/latch>
    * However, as of now, <latch> is part of the **standard** after `C++20`, so you can include <latch> directly with your C++20 compiler
- `std::latch` object should be **constructed** with a specified **count**, and each call to `.count_down()` decrements this count
- When the **count** reaches `zero`, the `std::latch` is considered `ready`
- `Any thread` that calls `.wait()` on the `std::latch` will **block** until the **count** reaches `zero`
- Because `std::latch` **cannot be copied**, you should **pass** the `reference to std::latch` for `other threads`

### `std::barrier`
A `std::barrier` is **similar** to a `std::latch` in that they **block** threads based on a **count** value
- However, it's **different** from a `std::latch` in that
    * The **count** is **decremented only once** by calling `arrive_and_wait()` on `each thread`
        + This is because `arrive_and_wait()` **decrements** the count and then reaches the barrier to **block** the `thread` **right away** until `all threads` reach the barrier
        + With a `std::latch`, it's **possible** to **decrement** the count **multiple** times in a `same thread`
    * A `std::barrier` is **reusable** across multiple cycles
        + Technically speaking, there's **another method to decrement** the count
            - call `arrive_and_drop()`
        + This method not only **decrements** the count but also **leaves** from the cycle, resulting in the next cycle's count being one less than the previous one
        + Therefore, once the `std::barrier` object's count reaches `zero`, it can be **reused** for subsequent cycles with the **modified count** value based on the calls of `arrive_and_drop()`
- The code below shows the example with `std::barrier` object
    ```cpp
    #include <barrier>  // Include header as well

    void process_data(data_source& source, data_sink& sink) {
        std::vector<ThreadGuard> threads;
        std::vector<DataChunk> chunks;
        ResultBlock result;

        // Arbitrary number of threads to use
        unsigned const numberThreadsInUse = std::max<int>(std::thread::hardware_concurrency() - 10, 5);    
        std::barrier sync(numberThreadsInUse);                                                         

        for (unsigned i = 0; i < numberThreadsInUse; ++i) {
            threads.emplace_back(std::thread([&, i] {               // Pass barrier as reference
                while (!source.done()) {
                    // Only first thread reads and divides data
                    if (!i) {  
                        DataBlock currentBlock = source.getNextDataBlock();
                        chunks = divideIntoChunks(currentBlock, numberThreadsInUse);   
                    }

                    sync.arrive_and_wait();  
                    // convert processed data chucks into a complete data block
                    result.buildFromChunks(i, numberThreadsInUse, process(chunks[i]));  

                    sync.arrive_and_wait();  
                    // Only first thread writes the result
                    if (!i)
                        sink.writeData(std::move(result));  
                }
            }));
        }
    }
    ```
- Just like `std::latch`, `std::barrier` objects are **non-copyable**
    * You must pass them by **reference** to `threads` to ensure **proper synchronization** across `multiple threads`


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}