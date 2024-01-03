---
title: "Effective Modern C++ : Chapter 1"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective Modern C++, auto, decltype, Type Deduction, Function Template]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-1
---

# Deducing Types

> 이 포스트는 Effective Modern C++(1st Edition)를 바탕으로 작성되었습니다.

## Item 1

### Understand template type deduction
An understanding how **template type deduction** works can be easily achieved by understanding type deduction rules of **function templates**
```c++
// general form for function templates
template <typename T>
void f(ParamType param);

f(expr);
```
- during compilation, compilers use `expr` to deduce 2 types
    * one for `T`
        + it's natural to expect that the type deduced for `T` is the same as the type of the argument passed to the function
        + but it doesn't always work that way
        + the type deduced for `T` is dependent not just on the type of `expr`,
            - but also on the form of `ParamType`
    * and one for `ParamType`
        + there are 3 cases where different deduction rules are applied to

### Case 1
Case 1: `ParamType` is a Reference or Pointer, but not a Universal Reference
```c++
// example 1
template <typename T>
void f(T &param);       // param is a reference

int x = 25;             // x is an int
const int cx = x;       // cx is a const int
const int &x = x;       // rx is a reference to x as a const int

f(x);                   // T is int; param's type is int&
f(cx);                  // T is const int; param's type is const int&
f(rx);                  // T is const int; param's type is const int&


// example 2
template <typename T>
void f(const T &param);       // param is new a reference-to-const

int x = 25;                   // as before
const int cx = x;             // as before
const int &x = x;             // as before

f(x);                         // T is int; param's type is const int&
f(cx);                        // T is int; param's type is const int&
f(rx);                        // T is int; param's type is const int&


// example 3
template <typename T>
void f(T *param);               // param is now a pointer

int x = 25;                     // as before
const int *px = &x;             // px is a pointer to x as a const int
const int * const cpx = &x;     // cpx is a constant pointer to x as a const int

f(&x);                          // T is int; param's type is int*
f(px);                          // T is const int; param's type is const int*
f(cpx);                         // T is const int; param's type is const int*; top-level const-ness is ignored
```
- in this case, type deduction works like this:
    1. if `expr`'s type is a reference, **ignore** the reference part
        + same rules are applied to pointers as well
    2. then pattern-match `expr`'s type against `ParamType` to determine `T`
- it's worth noting that the same rule is applied to this case as the [**copy operation**](https://sadoe3.github.io/cpp/primer-chapter2/#pointer-with-const) we've covered through Primer
    * top-level `const`-ness is ignored
    * low-level `const`-ness matters
- you can think of a Universal Reference as rvalue reference
    * it's **Universal** because
        + if we pass lvalue, then it becomes an lvalue reference
        + if we pass rvalue, then it remains as an rvalue reference 
    * it's related to the concept of [**reference collapsing**](https://sadoe3.github.io/cpp/primer-chapter16/#template-argument-deduction) which we've covered through Primer

### Case 2
Case 2: `ParamType` is a Universal Reference
```c++
template <typename T>
void f(T &&param);            // param is now a universal reference

int x = 25;                   // as before
const int cx = x;             // as before
const int &x = x;             // as before

f(x);                         // x is lvalue, so T is int&, param's type is also int&
f(cx);                        // cx is lvalue, so T is const int&, param's type is also const int&
f(rx);                        // rx is lvalue, so T is const int&, param's type is also const int&
f(25);                        // 25 is rvalue, so T is int, param's type is therefore int&&
```
- in this case, type deduction works like this:
    * if `expr` is an **lvalue**, both `T` and `ParamType` are deduced to be **lvalue** references
    * if `expr` is an **rvalue**, the "normal" (i.ei., Case 1) rules are applied to

## Case 3
Case 3: `ParamType` is Neither a Pointer nor a Reference
```c++
template <typename T>
void f(T param);            // param is now passed by value

int x = 25;
int cx = x;
int& rx = x;
const int& rcx = x;
int* px = &x;
int* const cpx = &x;
const int* const cpcx = &x;

f(x);                        // T's and param's types are both int
f(cx);                       // T's and param's types are still both int
f(rx);                       // T's and param's types are still both int
f(rcx);                      // T's and param's types are still both int
f(px);                       // T's type is int; param's type is int*
f(cpx);                      // T's type is int; param's type is int*
f(cpcx);                     // T's type is const int; param's type is const int*
```
- if it's pass-by-value, then the `param` will be a **copy** (a completely new object) of whatever is passed in
- in this case, type deduction works like this:
    1. if `expr`'s type is a **reference**, **ignore** the reference part
        + however, it's worth noting that if `expr`'s type is a **pointer**, **respect** the pointer part
            - because compilers regard a pointer as a copyable type
        + but as you know, it's not possible for references to be copied
        + moreover, if normal object (not reference nor pointer) is passed, its `const`ness is **ignored** as well
    2.  if `expr`'s **reference**-ness is ignored, then **ignore** its low-level `const`-ness
        + if `expr`'s type is a **pointer**, then **ignore** its **top**-level `const`-ness only, and **respect** its **low**-level `const`-ness
        + same rules are applied to `volatile` qualifiers

### Array Argument
If the type passed to is an **array**, there are 2 possible results
```c++
const char name[] = "Kyle";     // name's type is const char[5]

// case 1
template <typename T>
void f(T param);                // param is now passed by value

f(name);                        // param's type is const char*

// case 2
template <typename T>
void f(T &param);               // param is now a reference

f(name);                        // param's type is const char (&) [5]
```
- if `ParamType` is `T`, then normal **array-to-pointer** decay rule is applied
    * this is what you'd expect
- however, if `ParamType` is `T&`, the deduced type is a **reference to an array** with exact size
    * note that the size of an array is deduced as well
    * hence, we can implement a function template which returns the size of the given array:
        ```c++
        template <typename T, std::size_t N>
        constexpr std::size_t getArraySize(T(&)[N]) noexcept {
            return N;
        }
        ```
        + `constexpr` makes its result available during compilation
        + `noexcept` helps compilers generate better code
            - it's covered in [**Item 14**]()


## Item 2

### Understand `auto` type deduction
O


## Item 3

### Understand `decltype`
O


## Item 4

### Know how to view deduced types
- Deduced types can often be seen using
    * IDE editors
    * compiler error messages
    * the Boost TypeIndex library
- however, the results of some tools may be neither helpful nor accurate
	* hence, an understanding of C++’s **type deduction** rules remains essential



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}