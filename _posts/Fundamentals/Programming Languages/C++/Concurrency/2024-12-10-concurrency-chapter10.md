---
title: "C++ Concurrency : Parallel Algorithms in STL"

categories:
    - cpp

tags:
    - [C++, Programming Language, Concurrency, STL, parallel algorithm, execution policy]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-12-10
---

# C++ Concurrency : Chapter 10

> 이 포스트는 `C++ Concurrency in Action (2nd ed.)`을 바탕으로 작성되었습니다.

## Parallel Algorithms
The C++17 standard introduced the concept of **parallel algorithms** to the C++ Standard Library
```cpp
std::vector<int> collection;
// normal
std::sort(collection.begin(). collection.end());
// parallel
std::sort(std::execution::par, collection.begin(). collection.end());
```
- Parallel algorithms are **overloads** of many functions that operate on `ranges`
- The `parallel` versions have the **same** `signature` as their `single-threaded` counterparts
    * The only **difference** is the addition of a **new** `first parameter`
        + which specifies the **`execution policy`**
- It's important to note that the `execution policy` is a **permission**, **not** a **requirement**
    * This means that the library **may** still execute the code on a `single thread`, even if an `execution policy` is provided

### Execution Policies
There are **3** `execution policies` defined in the `<execution>` header file
- **`std::execution::sequenced_policy`**
    * Use this policy if you want to **disable** `parallelism`
        + It **guarantees** that the implementation will perform all operations on the thread that called the function
        + Additionally, it **forces** the library to execute operations in the `exact order` as specified in the program
            - In `C++17` (on which the textbook was based), the order might **vary** in certain cases
            - However, in `C++20` and later, the order is **guaranteed** when using `std::execution::seq`
    * This policy **guarantees** `serial execution`, unlike the other two policies
    * Use the `std::execution::seq` object to apply this policy
- **`std::execution::parallel_policy`**
    * Use this policy if you want to achieve `parallelism` with **general-purpose** parallelism
    * The **precise** `order of execution` is **unspecified**, but it is **guaranteed** that **all operations** performed on a given thread will occur in a `definite order`
        + Therefore, these operations **can** use **synchronization mechanisms**
            - such as locks, condition variables
    * Use the `std::execution::par` object to apply this policy
- **`std::execution::parallel_unsequenced_policy`**
    * Use this policy for `parallelism` with **maximum performance**
        + It is best suited for computationally heavy tasks
            - such as numerical algorithms, image processing
    * The `order of execution` is **not specified**
        + Therefore, it's crucial that the operations are
            - **Independent**
            - **Free from synchronization mechanisms**
    * Use the `std::execution::par_unseq` object to apply this policy
- It's important to note that you **cannot** define **custom** `execution policies`
    * Thus, you should use the objects provided by the library instead.
- Most algorithms from the `<algorithm>` and `<numeric>` headers have overloads that accept an `execution policy`


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}