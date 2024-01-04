---
title: "Effective Modern C++ : Chapter 3"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective Modern C++, nullptr, using, scoped enum, deleted method, override, const_iterator, noexcept, constexpr]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-4
---

# Moving to Modern C++

> 이 포스트는 Effective Modern C++(1st Edition)를 바탕으로 작성되었습니다.

## Item 7

### Distinguish between () and {} when creating objects
O


## Item 8

### Prefer `nullptr` to `0` and `NULL`
O


## Item 9

### Prefer alias declarations to `typedef`s
O


## Item 10

### Prefer scoped `enum`s to unscoped `enum`s
X


## Item 11

### Prefer deleted functions to `private` undefined ones
O


## Item 12

### Declare overriding functions `override`
X


## Item 13

### Prefer const_iterators to iterators
O


## Item 14

### Declare functions `noexcept` if they won't emit exceptions
O


## Item 15

### Use `constexpr` whenever possible
X

## Item 16
### Make `const` member functions thread safe
Make `const` member functions thread safe unless you’re certain they’ll never be used in a concurrent context
- use of `std::atomic` variables may offer better performance than a mutex, but they’re suited for manipulation of only a single variable or memory location


## Item 17

### Understand special member function generation
O -> need tests for the last part -> regarding member function templates




[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}