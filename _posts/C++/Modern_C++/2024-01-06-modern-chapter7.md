---
title: "Effective Modern C++ : Chapter 7"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective Modern C++, Concurrency]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-6-3
---

# The Concurrency API

> 이 포스트는 Effective Modern C++(1st Edition)를 바탕으로 작성되었습니다.

## Concurrency Study Plan
1. Clean Code
    * chapter 13
    * appendix (not mandatory)
2. Concurrency in Action
3. Effective Modern C++
    * chapter 7
4. C++17: coroutines

## Item 35
### Prefer task-based programming to thread-based

## Item 36
### Specify std::Launch::async if asynchronicity is essential

## Item 37
### Make std::threads unjoinable on all paths

## Item 38
### Be aware of varying thread handle destructor behavior

## Item 39
### Consider void futures for one-shot event communication

## Item 40
### Use `std::atomic` for concurrency, volatile for special memory


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}