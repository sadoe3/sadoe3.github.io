---
title: "Effective Modern C++ : Chapter 4"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective Modern C++, Smart Pointer, std::shared_ptr, std::unique_ptr]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-5
---

# Smart Pointers

> 이 포스트는 Effective Modern C++(1st Edition)를 바탕으로 작성되었습니다.

## Item 18

### Use `std::unique_ptr` for exclusive-ownership resource management
As we've convered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter12/#smart-pointers)
- `std::unique_ptr` is a small, fast, move-only smart pointer for managing resources with exclusive-ownership semantics
- by default, resource destruction takes place via `delete`, but custom deleters can be specified
	* stateful deleters and function pointers as deleters increase the size of `std::unique_ptr` objects
- converting a `std::unique_ptr` to a `std::shared_ptr` is easy


## Item 19

### Use `std::shared_ptr` for shared-ownership resource management
As we've convered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter12/#smart-pointers)
- `std::shared_ptr`s offer convenience approaching that of garbage collection for the shared lifetime management of arbitrary resources
- compared to `std::unique_ptr`, `std::shared_ptr` objects are typically twice as big, incur overhead for control blocks, and require atomic reference count manipulations
- default resource destruction is via `delete`, but custom deleters are supported
- the type of the deleter has no effect on the type of the `std::shared_ptr`
- avoid creating `std::shared_ptr`s from variables of raw pointer type


## Item 20

### Use `std::weak_ptr` for `std::shared_ptr`-like pointers that can dangle
As we've convered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter12/#weak_ptr)
- the potential use cases for `std::weak_ptr` include
    * caching
    * observer lists
    * the prevention of `std::shared_ptr` cycles


## Item 21

### Prefer `std::make_unique` and `std::make_shared` to direct use of new
As we've convered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter12/#make_shared-function)
- Compared to direct use of `new`, `make function`s eliminate source code duplication improve exception safety, and, for `std::make_shared` and `std::allocate_shared`, generate code that's smaller and faster.
- situations where use of `make function`s is inappropriate include the need to specify custom deleters and a desire to pass braced initializers
- for `std::shared ptr`s, additional situations where `make function`s may be ill-advised include
	* classes with custom memory management and
	* systems with memory concerns, very large objects, and `std::weak_ptr`s that outlive the corresponding `std::shared_ptr`s


## Item 22

### When using the Pimpl Idiom, define special member functions in the implementation file
The *Pimpl* (pointer to implementation) *Idiom* **decreases build times** by reducing compilation dependencies between class clients and class implementations
```c++
// in "widget.h"
class Widget {
public:
// other members
    Widget();                               // declarations
    ~Widget();                              // only
    Widget(const Widget &rhs);              // declarations
    Widget& operator=(const Widget &rhs);   // only
    // other members
private:
    struct Impl;                            // declarations
    std::unique_ptr<Impl> pImpl;            // only
};


// in "widget.cpp"
#include "widget.h"
// other codes
struct Widget::Impl { std::string name; /* other data members */ };

Widget::Widget() : pImpl(std::make_unique<Impl>())
{}
Widget::~Widget() = default;                // use default definition

Widget::Widget(const Widget &rhs) : pImpl(nullptr) {
    if(rhs.pImpl)
        pImpl = std::make_unique<Impl>(*rhs.pImpl);
}
Widget& operator=(const Widget &rhs) {
    if(!rhs.pImpl)
        pImpl.reset();
    else if(!pImpl)
        pImpl = std::make_unique<Impl>(*rhs.pImpl);
    else
        // if both objects have allocated their own memory for data members;
        // assign the implementation part only
        *pImpl = *rhs.pImpl;                 
    return *this;
}
```
- for `std::unique_ptr` `pImpl` pointers, **declare** special member functions in the class **header**, but **implement** them in the **implementation file**
    * do this even if the default function implementations are acceptable
- the above advice applies to `std::unique_ptr` but **not** to `std::shared_ptr`
    * generally, everything would compile and run as we'd hope


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}