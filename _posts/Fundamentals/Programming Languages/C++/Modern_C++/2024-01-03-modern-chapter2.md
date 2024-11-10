---
title: "Effective Modern C++ : Chapter 2"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective Modern C++, auto]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-3
---

# auto

> 이 포스트는 Effective Modern C++(1st Edition)를 바탕으로 작성되었습니다.

## Item 5

### Prefer `auto` to explicit type declaration
- `auto` is useful
    * if you want to **force the initialization**, or
    * if you want to write robust code that handles **type mismatches**
        + which can lead to portability or efficiency problems
    * if you want to ease the process of **refactoring**
        + for instance, if a function is declared to return an `int`, but you later decide that a `long` would be better
            - then, the calling code **automatically updates** itself the next time you compile
        + otherwise, you'll need to find all the call sites so that you can revise them one by one 
    * if you want to type less than what's expected
- `auto`-typed objects are subject to the **pitfalls** described in [**Item 2**](https://sadoe3.github.io/cpp/modern-chapter1/#item-2) and [**Item 6**](https://sadoe3.github.io/cpp/modern-chapter2/#item-6)


## Item 6

### Use the explicitly typed initializer idiom when `auto` deduces undesired types
```c++
std::vector<bool> features(const Widget& w);    // returns the copy of std::vector<bool>

Widget w;
... // some codes

// case 1
bool highPriority = features(w)[5];             
processWidget(w, highPriority);                 // ok;

// case 2
auto highPriority = features(w)[5];             
processWidget(w, highPriority);                 // undefined behavior;
```
- because `std::vector` has template specialization for type `bool`, its `operator []` returns `std::vector<bool>::reference` instead of `bool&`
    * for other types, it returns `Type&`
    * the reason why the specialization exists is because `std::vector<bool>` stores its `bool` elements in packed form, one bit per `bool`
        + however, C++ **forbids** references to bits
    * hence, `std::vector<bool>` has `std::vector<bool>::reference` which is a nested class that is implemented to point to that element (bit) and has **implicit conversion** to `bool`
        + therefore, `std::vector<bool>::reference` objects must be usuable in essentially all contexts where `bool&`s can be
        + note that it's not `bool&` but `bool`
- the deduced type for `auto` depends on how `std::vector<bool>::reference` is implemented
    * one implementation is for such objects to contain a **pointer** to that memory
        + and this is the case of `std::vector<bool>::reference`
    * `features(w)` returns **temporary** `std::vector<bool>` object and `[] opeartor` returns `std::vector<bool>::reference` which points to the **element from it**
        + at the end of the statement, that **temporary** object is **destroyed**
        + this makes `highPriority` contains a **dangling** pointer
        + therefore, the second call to `processWidget` has **undefined behavior**
- `std::vector<bool>::reference` is an example of a **proxy** class
    * a class that exists for the purpose of **emulating** and augmenting the behavior of some **other type**
    * the Standard Library's smart pointer types are also proxy classes
    * some proxy classes are designed to be apparent to clients
        + that's the case for `std::shared_ptr`, and `std::unique_ptr`
        + it's okay for `auto` to deduce such types
    * other proxy classes are designed to act more or less invisibly
        + `std::vector<bool>::referecne` and `std::bitset::reference` are the examples of the **invisible** proxy classes
        + as a general rule, it's unusual for `auto` to deduce such types

### The explicitly typed initializer idiom
If `auto` is **not deducing** the type you want it to deduce, try to **force** it to deduce a certain type
```c++
auto highPriority = static_cast<bool>(features(w)[5]);             // refined use; uses explicit conversion

bool highPriority = features(w)[5];                                // still ok; uses implicit conversion
```
- the author of the textbook called this way as *the explicitly typed initializer idiom*
    * try to use this way if you want to express that "I'm intentionally casting to different type"
- still, the second version is acceptable
    + although it hardly announces that "I know the type resulting from `features(w)[5]` is not `bool`, so I'm intentionally casting it to `bool`


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}