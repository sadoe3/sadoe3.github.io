---
title: "Effective C++ : Chapter 3"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective C++, Dynamic Memory, C++17]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-28
---

# Resource Management

> 이 포스트는 Effective C++(3rd Edition)를 바탕으로 작성되었습니다.

## Item 13

### Use objects to manage resources
The idea of using objects to manage resources is often call *Resource Acquisition Is Initialization* (**RAII**)
- to prevent resource leaks, try to use **RAII** objects, such as **smart pointers**, that acquire resources in their constructors and release them in their destructors


## Item 17

### Store `new`d objects in smart pointers in standalone statements
Consider the following code
```c++
functionName(std::unique_ptr<T>(new T), otherFunction());
```
- there are 3 operations to be done to call `functionName()`
    * execute `new` T
    * call `std::unique_ptr`'s constructor
    * call `otherFunction()`
- however, prior to C++17, the order of evaluations for each parameter by C++ compilers is **undefined**
    * this is different from the way languages like Java or C# work, where function parameters are always evaluated in a particular order
    * which means that the call to `otherFunction()` can be made before executing `new` or calling the constructor
        + of course, it's guaranteed for the constructor to be called after executing `new` operation
    * the problem is that if the call to `otherFunction()` is made before calling the constructor
        + and the `otherFunction()` throws an exception
        + there would be a memory leak because the destructor won't be called because the smart pointer is not constructed
- there are 3 solutions to this problem
    * store `new`d objects in smart pointers in standalone statements
            ```c++
            std::unique_ptr<T> pw(new T);
            functionName(pw, otherFunction());
            ```
        + this is the solution from the textbook
        + however, the 2 solutions below are preferred
    * call `std::make_unique()`
        + this solution works for the compiler which supports prior versions to C++17
        + by wrapping the allocation phase and construction phase, there's no chance for `otherFunction()` to be called before executing `new` operation
    * use compiler which supports C++17
        + with C++17, the order of evaluations is now **unspecified**
        + which means that the second parameter may still be evaluated before the first one
        + however, it's guaranteed that any parameter is fully evaluated before the evaluation for the next parameter is started
        + for instance,
            - `f(a(x), b, c(y));`
        + if the compiler chooses to evaluate `x` first, then it must evaluate `a(x)` before processing `b`, `c(y)` or `y`


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}