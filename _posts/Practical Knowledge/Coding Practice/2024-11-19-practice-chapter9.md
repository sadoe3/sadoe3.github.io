---
title: "Clean Code : Chapter 13"

categories:
    - coding-practice

tags:
    - [Clean Code, Coding Practice, Concurrency]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-19
---

# Concurrency

> 이 포스트는 Clean Code(1st Edition)를 바탕으로 작성되었습니다.

## Why Concurrency?

### Single-Threaded Applications
In the world of `single-threaded` application, you need to **wait** to get something until it gets **done**
- this type of application works for most cases
- however, in some cases
    * like, handling multiple requests **quickly**, or executing two or more processes **simultaneously**
- it **loses the performance** or it's **impossible** to even implement it in a `single-threaded` world

### Multi-Threaded Applications
You can resolves these issues by implementing a **`multi-threaded`** application
- concurrency **decouples** `what we want to get` from `when it gets done`
- however, writing a `clean` concurrent program is **difficult** bceause it is **easy** to write multi-threaded code that looks `good on the surface` but is `broken at a deeper level`
    * such code **works** as you expect **until** the system is placed under **stress**


## Concurrency Defense Principles
The recommendations below are the `techniques` and `principles` to **defend** your program from the **problems** of concurrent code

### Single Responsibility Principle
*Keep your concurrency-related code separate from other code*
- it's too **common** for `concurrency implementation` details to be **embedded directly** into other production code
- this is **not recommended** because
    * `concurrency-related code` has its **own life cycle** of *development*, *change*, and *tuning*
    * it has its **own challenges** which are *different* from and often *more difficult* than noncocurrency-related code
- once you correctly separate, then that threaded code is **pluggable**
    * meaning that it can be run in **serveral configurations**

### Get Your Nonthreaded Code Working First
*Do not try to track nonthreading bugs and threading bugs simultaneously*
- when you start to implement threaded code
    * it's recommended to write the **nonthreaded** version **first** so that you can check whether your algorithm works or not
- otherwise, you may face the situation where you track down the threading bugs only because it is related to your threaded code
    * although the cause of bugs doesn't related to concurrency

### Limit the Scope of Data
*Limit the moments where any data that may be shared*
- although it's possible to share any data **safely** if you follows the **rules** correctly
    * it's highly recommended to **limit** the `access of shared data`
- because the more places shared data can get updated, the more you need to **write code** to make that sharing safe
    * this addtional code might lead to your mistake to forget to protect those places
    * or code duplication

### Read-Only Situation
*Use references to constants or copies when you need to read only the shared data*
- if you don't need to write the shared data, get the help from the **compiler** to check whether you actually don't write or not

### Data Partitioning
*Try to partition data into independent subsets based on the requests*
- you can **minimize concurrent problems**, and enable better resource utilization by partitioning data into independent subsets and passing them to the independent thread or processor

### Critical Blocks and Sections
*Minimize the Time for Exclusive Use*
- you utilize `std::mutex` to exclusively execute a certain code block
- it's highly recommended to **minimize** the `scope of locked code`
    * otherwise, other threads will **wait**
    * and this leads to **performance decrease**

### Tests
*Write tests that have the potential to expose problems and then run them frequently, with different programtic configurations and system configurations and load*
- **do not ignore** the `failure` just because
    * the test passes on a subsequent run
    * or it happens very rarely

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}