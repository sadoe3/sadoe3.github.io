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
A novel feature of **braced initialization** is that it **prohibits** implicit narrowing converions (which causes data loss) among built-in types
```c++
double x, y, z;
/// some codes
int sum { x+y+z };       // error; trying narrowing conversion

// example of vexing parse
ClassName c1(10);           // call ClassName constructor with argument 10
ClassName c1();             // most vexing parse!; declare a function

ClassName c1{};             // call ClassName default constructor
ClassName c1;               // call ClassName default constructor
ClassName c1 = ClassName(); // call ClassName default constructor

// example of std::initializer_list issue
class ClassName {
public:
    ClassName(int i, bool b);
    ClassName(int i, double d);
    ClassName(std::initializer_list<long double> il);
    ClassName() = default;
    // other members
};
// some codes
ClassName c1(10, true);             // call first constructor
ClassName c2{10, true};             // call third constructor
ClassName c3(10, 5.0);              // call second constructor
ClassName c4{10, 5.0};              // call third constructor
ClassName c5{};                     // call default constructor
ClassName c6({});                   // call third constructor


``` 
- another characteristic of **braced initalization** is that it's immune to C++'s **most vexing parse**
    * a side effect of C++'s rule that anything that can be parsed as a declaration must be interpreted as one
    * the typical example of this is that calling the default constructor is actually declaring a function instead
- during constructor overload resolution, **braced initializers** are matched to `std::initializer_list` parameters as its **first priority**, although other consturctors offer seemingly better matches
    * compilers fall back on normal overload resolution
        + *only if* there's **no way to convert** the types of the arguments in a braced initializer to the type in a `std::initializer_list`
- an example of where the choice between parentheses and braces can make a significant different is
    * creating a `std::vector<numeric type>` with two arguments


## Item 8

### Prefer `nullptr` to `0` and `NULL`
`nullptr`'s advantage is that it doen'st have an integral type
- to be honest, it doen'st have a pointer type either
    * its actual type is `std::nullptr_t` which implicitly converts to all raw pointer types so that it makes `nullptr` act as if it were a pointer of all types
- the fact that template type deduction  deduces the **wrong** types for `0` and `NULL`
    * is the most compelling reason to use `nullptr` instead of them


## Item 9

### Prefer alias declarations to `typedef`s
Alias declarations may be **templatized**, while `typedef`s cannot
```c++
// using
template <typename T>
using MyAllocList = std::list<T, MyAlloc<T>>;           // ok;

template <typename T>
class Widget {
private:
    MyAllocList<T> list;                                    
// other members
};



// typedef
template <typename T>
typedef std::list<T, MyAlloc<T>> MyAllocList;           // error; typedef doesn't support templatization

template <typename T>
struct MyAllocList {
    typedef std::list<T, MyAlloc<T>> type;              // now it's ok;
};

template <typename T>
class Widget {
private:
    typename MyAllocList<T>::type list;                      
// other members
};
```
- alias templates avoid the `::type` suffix and, in templates, the `typename` prefix often required to refer to `typedef`s
    ```c++
    std::remove_const<T>::type              // C++11: const T -> T
    std::remove_const_t<T>                  // C++14 equivalent

    std::remove_reference<T>::type          // C++11: const T&/T&& -> T
    std::remove_reference_t<T>              // C++14 equivalent

    // how it works
    template <class T>
    using remove_const_t = typename remove_const<T>::type;

    template <class T>
    using remove_reference_t = typename remove_reference<T>::type;
    ```
    * C++14 offers alias templates for all the C++11 type traits transformations
    

## Item 10

### Prefer scoped `enum`s to unscoped `enum`s
As we've convered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter19/#enumerations)
- scoped `enum`s are visible only whihin the `enum`
    * they convert to other types **only** with a cast


## Item 11

### Prefer `delete`d functions to `private` undefined ones
As we've convered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter13/#default-member-functions)
- we can prevent the default method from being created by the compiler by using `=delete`
- moreover, any function my be `delete`d, including non-member functions and template instantiation
    ```c++
    // non-member function
    bool isLucky(int number);
    bool isLucky(char) = delete;            // reject chars
    bool isLucky(bool) = delete;            // reject bools
    bool isLucky(double) = delete;          // reject doubles and floats

    isLucky(true)            // error!



    // template instantiation
    // function template
    template<typename T>
    void func(T *ptr);

    template<>
    void func(void *ptr) = delete;
    template<>
    void func(const char *ptr) = delete;

    func("haha");           // error!

    // class template
    class ClassName {
    public:
        template <typename T>
        void func(T *ptr) {}
        // other members
    private:
        // error; specialization cannot happen at the class scope
        template <>                 
        void func(void *ptr);
    };

    class ClassName {
    public:
        template <typename T>
        void func(T *ptr) {}
        // other members
    };
    template <>                 
    void ClassName::func(void *ptr) = delete;       // ok;
    ```
    * note that if you want to `delete` the specialization of the nested class, you need to write the specialization at namespace scope


## Item 12

### Declare overriding functions `override`
As we've covered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter15/#virtual-method)
- it increases the readability of the code
- moreover, it makes the compilers **check** whether it's actually **overriding** or **hiding**
    ```c++
    class Base {
    public:
        virtual int func(double x) = 0;
    };

    class Derived1 : public Base {
    public:
        int func(double x) override { /* some definition */ };
    };

    class Derived2 : public Base {
    public:
        int func(int x) override { /* some definition */ };         // error; trying hiding
    };
    ```
    * without `override`, the code above would be compiled with `Derived2` class which **hides** `func` **instead of overriding**


## Item 13

### Prefer `const_iterator`s to iterators
As we've covered through [**Primer**](https://sadoe3.github.io/cpp/effective-chapter1/#use-const-whenever-possible)
- try to use `const_iterator`s any time you need an iterator,
    * yet have **no need to modify** what the iterator points to
- in maximally generic code
    * perfer non-member versions of `begin`, `end`, `rbegin`, etc., over their member function countparts


## Item 14

### Declare functions `noexcept` if they won't emit exceptions
Using `noexcept` permits compilers to generate **better** object code
```c++
RetType func(params) noexcept;          // C++11 style; most optimizable
RetType func(params) throw();           // C++98 sytle; less optimizable
RetType func(params);                   // less optimizable
```


## Item 15

### Use `constexpr` whenever possible
As we've covered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter2/#constexpr)
- `constexpr` [**functions**](https://sadoe3.github.io/cpp/primer-chapter6/#constexpr-functions) can produce compile-time results
    * when called with arguments whose values are known during compilation


## Item 16
### Make `const` member functions thread safe
Make `const` member functions thread safe unless you’re certain they’ll never be used in a concurrent context
- use of `std::atomic` variables may offer better performance than a mutex, but they’re suited for manipulation of only a single variable or memory location 


## Item 17

### Understand special member function generation
As we've covered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter13/#copy-control)
- **member function template**s never suppress generation of speicalized member functions except for the default constructor
    ```c++
    class ClassName {
    public:
        ClassName() = default;
        template <typename T>
        ClassName(const T& rhs) { std::cout << "template version" << std::endl; }
    };
    // some codes
    ClassName a;
    ClassName b(a);             // prints nothing because it calls the synthesized version
    ClassName c(1);             // prints template version
    ```
    + this rule is applied to copy/move constructors and assignment operators


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}