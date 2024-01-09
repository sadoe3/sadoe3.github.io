---
title: "Clean Code : Chapter 5"

categories:
    - coding-practice

tags:
    - [Clean Code, Coding Practice, Formatting]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-9
---

# Formatting	

> 이 포스트는 Clean Code(1st Edition)를 바탕으로 작성되었습니다.

## Vertical Formatting

### Vertical Density
Lines of code that are **related** should appear **vertically dense**

### Vertical Spacing
A function consists of multiple groups of lines which **respectively** represent a complete **thought**
- those **thoughts** should be **separated** from each other with **blank lines** 

### Vertical Ordering
In general we want function call dependencies to point in the **downward** direction
- that is, a function that is **called** should be **below** a function that does the **calling**
    * this creates a nice flow down the source code **from high level to low level**


## Indentation
The **indendation** is necessary in order to increase the **readability** of the source file
```c++
// bad
namespace std { template <typename T> void swap(T& a, T& b) { T temp(a); a = b; b = temp; } }

// clean
namespace std {
    template <typename T>
    void swap(T& a, T& b) {
        T temp(a);
        a = b;
        b = temp;
    }
}
```


## Team Project
A **team** of developers should **agree** upon a single formatting style, and a single naming rule
- so that the **final code** would have a **consistent** and smooth style


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}