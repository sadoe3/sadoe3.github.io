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

date: 2024-1-2
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
    * it's related to the concept of [**reference collapsing**](https://sadoe3.github.io/cpp/primer-chapter16/#template-argument-deduction-and-references) which we've covered through Primer

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

### Case 3
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
        ... // some codes
        int a[] = {1,2,3};
        double b[getArraySize(a)];  // equivalent to double b[3];
        ```
        + `constexpr` makes its result available during compilation
        + `noexcept` helps compilers generate better code
            - it's covered in [**Item 14**](https://sadoe3.github.io/cpp/modern-chapter3/#item-14)

### Function Arguments
**Function types** can decay into **function pointers**, and everything we've discussed regarding type deduction for **arrays** applied to type deductions and their decay into **function pointers** as well
```c++
void someFunc(int, double);     // type is void(int, double)

template <typename T>
void f1(T param);               // param passed by value

template <typename T>
void f2(T &param);              // param passed by reference

f1(someFunc);                   // param is deduced as pointer-to-function
                                // type is void (*)(int, double)
f2(someFunc);                   // param is deduced as reference-to-function
                                // type is void (&)(int, double)
```
- unlike array arguments, this rarely makes any difference in practice


## Item 2

### Understand `auto` type deduction
`auto` type deduction is **same** as **template** type deduction with one exception
```c++
// there are 3 cases for auto type deduction as well because it's template type deduction without one exception
auto x = 25;                    // case 3 (x is int)
const auto cx = x;              // case 3 (cx is const int)
const auto &rx = x;             // case 1 (rx is reference-to-const-int)

// case 2
auto&& rvr1 = x;                // x is int and lvalue; so rvr1's type is int&
auto&& rvr2 = cx;               // cx is const int and lvalue; so rvr2's type is const int&
auto&& rvr3 = 25;               // 25 is int and rvalue; so rvr3's type is int&&

// array arguments
const char name[] = "Kyle";     // name's type is const char[5]
auto arr1 = name;               // arr1's type is const char*
auto &arr2 = name;              // arr2's type is const char (&)[5]

// function arguments
void someFunc(int, double);     // type is void(int, double)
auto func1 = someFunc;          // func1's type is void (*) (int, double)
auto &func2 = someFunc;         // func2's type is void (&) (int, double)
```
- when a variable is declared using `auto`
    * `auto` plays the role of `T` in the template
    * and the **type specifier** for the object acts as `ParamType`
- the **one exception** is related to the initialization by a **braced initializer**
    ```c++
    // example of the exception
    auto x1 = 25;            // type is int, value is 27
    auto x2(25);             // same as above
    auto x3 = {25};          // type is std::initialier_list<int>, value is {27}
    auto x4{25};             // same as above



    // function template can't do this
    template <typename T>
    void f(T param);             

    f({11, 23, 7});          // error; can't deduce type for T
    // solution
    template <typename T>
    void f(std::initializer_list<T> param);             

    f({11, 23, 7});          // ok; T is deduce as int, ParamType is std::initializer_list<int>



    // 2 special cases where auto uses template deduction rules
    auto createInitList() {
        return {1,2,3};      // error; can't deduce type for {1,2,3}
    }

    std::vector<int> v;
    ... // some codes
    auto resetV = [&v](const auto &newValue) { v = newValue; };
    ... // some codes
    resetV({1,2,3});         // error; can't deduce type for {1,2,3}
    ```
    * `auto` deduces the object which is initialized by the braced initializer as `std::initializer_list<T>`
        + it's worth noting that `std::initializer_list` takes **single** type, which means that if there are values of **different types** in the braced initializer
            - `auto` **cannot deduce** the type 
            - then, the code would be rejected
    * function template **cannot deduce** the type of an object initialized by the braced initializer
    * it's worth noting that there are 2 cases where `auto` employs **template** type deduction rules
        + `auto` is utilized as the `return type` of a function
        + `auto` os utilized for the `parameter type` of a lambda


## Item 3

### Understand `decltype`
Typically, `decltype` *almost* always deduces what you'd expect
```c++
const int i = 0;                        // decltype(i) is const int

bool f(const Widget &w);                // decltype(w) is const Widget&
                                        // decltype(f) is bool(const Widget&)

struct Point {
    int x, y;                           // decltype(Point::x) is int
};                                      // decltype(Point::y) is not

Widget w;                               // decltype(w) is Widget

if(f(w)) {;}                            // decltype(f(w)) is bool

template <typename T>
class TemplateName {
    // other members
    T& operator[](std::size_t index);
    // other members
};
TemplateName<int> v;                    // decltype(v) is TemplateName<int>
if(v[0] == 0 ) {;}                      // decltype(v[0]) is int&
```
- as we've covered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter2/#decltype), it yields the deduced type **without any modifications**
    * however, rarely, `decltype` yields **surprsing** results
        ```c++
        int x = 0;

        decltype(x) a;      // a is int
        decltype((x)) b;    // b is int&
        ```
        + putting **parentheses around a name** can change the type that `decltype` reports for it 
        + for lvalue expressions of type `T` **other than names**,
            - `decltype` always reports a type of `T&`
            - `x` is name
            - `(x)` is not
- under C++14, there's an **alternative** to [**trailing return types**](https://sadoe3.github.io/cpp/primer-chapter6/#trailing-return-type)
    ```c++
    // C++11's trailing return type
    template <typename Container, typename Index>
    auto getElement(Container&& c, Index i) -> decltype(std::forward<Container>(c)[i]) {
        return std::forward<Container>(c)[i];
    }

    // C++14's decltype(auto)
    template <typename Container, typename Index>
    decltype(auto) getElement(Container&& c, Index i) {
        return std::forward<Container>(c)[i];
    }



    // example regarding the exception
    decltype(auto) f1() {
        int x = 0;
        return x;           // decltype(x) is int, so f1 returns int
    }
    decltype(auto) f1() {
        int x = 0;
        return (x);           // decltype((x)) is int&, so f1 returns int&
    }
    ```
    * `std::forward` is covered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter16/#template-argument-deduction-and-references)


## Item 4

### Know how to view deduced types
- Deduced types can often be seen using
    * IDE editors
    * compiler error messages
    * the Boost TypeIndex library
- however, the results of some tools may be neither helpful nor accurate
	* hence, an understanding of C++’s **type deduction** rules remains essential

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}