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
- `auto`-typed objects are subject to the **pitfalls** described in [**Item 2**]() and [**Item 6**](https://sadoe3.github.io/cpp/modern-chapter2/#item-6)


## Item 6

### Use the explicitly typed initializer idiom when `auto` deduces undesired types
O



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}