---
title: "Effective Modern C++ : Chapter 5"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective Modern C++, Rvalue, Move, Forward]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-5
---

# Rvalue References, Move Semantics, and Perfect Forwarding

> 이 포스트는 Effective Modern C++(1st Edition)를 바탕으로 작성되었습니다.

## Item 23

### Understand `std::move` and `std::forward`
As we've covered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter16/#template-argument-deduction-and-references)
- `std::move` performs an unconditional cast to an rvalue
- In and of itself, it doesn't move anything
- `std::forward` casts its argument to an rvalue only if that argument is bound to an rvalue
- neither `std::move` nor `std::forward` do anything at runtime
- move requests on `const` objects are treated as copy requests


## Item 25

### Use `std::move` on rvalue references, `std::forward` on universal references
As we've covered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter16/#template-argument-deduction-and-references)
- apply `std::move` to rvalue references and `std::forward` to universal references the last time each is used
- do the same thing for rvalue references and universal references being returned from functions that return by value
- never apply `std::move` or `std::forward` to local objects if they would otherwise be eligible for the return value optimization


## Item 26

### Avoid overloading on universal references
Overloading on universal references almost always leads to the **universal reference overloaed** being called **more frequently** than expected
```c++
std::multiset<std::string> names;

template <typename T>
void func(T &&name) {
    names.emplace(std::forward<T>(name));
}

std::string getNameFromIndex(int index);    // return name corresponding to index

void func(int index) {
    names.emplace(getNameFromIndex(index));
}

// some codes
std::string petName("ice");

func(petName);                      // these calls
func("Kyle");                       // all invoke the
func(std::string("Kyle"));          // T&& overload

func(24);                           // call int overload

short nameIndex = 3;
func(nameIndex);                   // error; call T&& overaload
                                    // because std::string doesn't have constructor
                                    // which takes short, it's an error 
```
- **perfect-forwarding constructors** are especially problematic
    ```c++
    class ClassName {
    public:
        // perfect forwarding constructor
        template <typename T>
        explicit ClassName(T &&n) : name(std::forward<T>(n)) {}
        // copy constructor
        ClassName(const ClassName &rhs) : name(rhs.name) {}
        // other members
    private:
        std::string name;
    };
    // some codes
    ClassName a("Kyle");
    auto cloneOfA(a);           // error; because a is not const object 
                                // perfect-forwarding constructor is called
                                // because it's the best-match and
                                // because std::string doesn't have constructor 
                                // which takes ClassName, it's an error
    const ClassName b("Kyle");
    auto cloneOfB(b);           // ok; call copy constructor
    ```
    * because they're typcially **better matches** than copy constructors for non-`const` lvalues

    ```c++
    class Base {
    public:
        // perfect forwarding constructor
        template <typename T>
        explicit Base(T &&n) : name(std::forward<T>(n)) {}
        // copy constructor
        Base(const Base &rhs) : name(rhs.name) {}
        // move constructor
        Base(Base &&rhs) : name(std::move(rhs.name)) {}

        // other members
    private:
        std::string name
    };

    class Derived : public Base {
    public:
        // copy constructor
        // but it calls base class's forwarding constructor
        // because instantiation from Derived is better match than the dynamic binding
        // and that call causes an error because std::string doesn't have a constructor
        // which takes Derived parameter
        Derived(const Derived &rhs) : Base(rhs) { }
        // move constructor
        // but it calls base class's forwarding constructor
        // same reason is applied
        Derived(Derived &&rhs) : Base(std::move(rhs)) { }

        // other members
    };
    ```

    * and they can **hijack** derived class calls to base class copy and move constructors 


## Item 27

### Familiarize yourself with alternatives to overloading on universal references
There are **alternatives** to the combination of universal references and overloading
- **using distinct function names**
    * so that we can avoid the problems related to overloading
    * this solution would be simple and easy to achieve 
- **passing parameters by lvalue-reference-to-`const`**
    * the drawback of this solution is that the design isn't as efficient as we'd prefer
    * however, giving up some efficiency to keep things simple might be a more attractive trade-off than it initially appeared 
- **passing parameters by value**
    ```c++
    class ClassName {
    public:
        // replace T&& constructor
        explicit ClassName(std::string n) : name(std::move(n)) {}
        // copy constructor
        ClassName(const ClassName &rhs) : name(rhs.name) {}

        // other members
    private:
        std::string name;
    };
    ```
    * through this way, you can still achieve what you want
- **using tag dispatch**
    ```c++
    // same codes
    std::multiset<std::string> names;

    std::string getNameFromIndex(int index) {
        return std::string("name" + index);
    }


    // key changes
    // non-integral arguments
    template <typename T>
    void funcImpl(T&& name, std::false_type) {
        names.emplace(std::forward<T>(name));
    }
    // integral argument
    void funcImpl(int index, std::true_type);

    template <typename T>
    void func(T&& name) {
        funcImpl(std::forward<T>(name), std::is_integral<std::remove_reference_t<T>>());
    }

    void funcImpl(int index, std::true_type) {
        func(getNameFromIndex(index));
    }


    // same codes
    std::string petName("ice");

    func(petName);                      // these calls
    func("Kyle");                       // all invoke the
    func(std::string("Kyle"));          // T&& overload

    func(24);                           // call int overload

    short nameIndex = 3;
    func(nameIndex);                    // now it calls int overload 
    ```
    * note that the integral overload version is shifted from `void func(int)` to `void funcImpl(int, std::true_type)`
- **using `std::enable_if`**
    ```c++
    class Base {
    public:
        // perfect forwarding constructor
        template <
            typename T, typename = std::enable_if_t<
                !std::is_base_of<Base, std::decay_t<T>>::value &&
                !std::is_integral<std::remove_reference_t<T>>::value
                >
            >
        explicit Base(T &&n) : name(std::forward<T>(n)) {}
        // copy constructor
        Base(const Base &rhs) : name(rhs.name) {}
        // move constructor
        Base(Base &&rhs) : name(std::move(rhs.name)) {}

        // other members
    private:
        std::string name;
    };

    class Derived : public Base {
    public:
        // copy constructor
        // now the dynamic binding version is the better-match 
        Derived(const Derived &rhs) : Base(rhs) { }
        // move constructor
        // now the dynamic binding version is the better-match 
        Derived(Derived &&rhs) : Base(std::move(rhs)) { }

        // other members
    };
    ```
- universal reference parameters often have **efficiency advantages**
    * but they typically have **usability disadvantages**


## Item 28

### Understand reference collapsing
As we've covered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter16/#template-argument-deduction-and-references)
- **reference collapsing** occurs in 4 contexts:
	* template instantiation
	* `auto` type generation
	* creation and use of `typedef`s, alias declarations
	* use of `decltype`
- when compilers generate a reference to a reference in a **reference collapsing** context, the result becomes a single reference
	* if either of the original references is an lvalue reference, the result is an lvalue reference
	* otherwise it's an rvalue reference
- universal references are rvalue references in contexts where type deduction distinguishes lvalues from rvalues and where **reference collapsing** occurs


## Item 29

### Assume that move operations are not present, not cheap, and not used
However, if you’re dealing with types or support for move semantics
- there’s no need for these assumptions


## Item 30

### Familiarize yourself with perfect forwarding failure cases
Perfect forwarding **fails** when template type deduction fails or when it deduces the wrong types
- the kinds of arguments that lead to perfect forwarding failure are
    * braced initializers
    * null pointers expressed as `0` or `NULL`
    * declaration-only integral `const static` data members
    * template and overladed function names
    * bitfields


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}