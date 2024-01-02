---
title: "Effective C++ : Chapter 1"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective C++, Preprocessor, const, static, Initialization]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-28
---

# Accustoming Yourself to C++

> 이 포스트는 Effective C++(3rd Edition)를 바탕으로 작성되었습니다.

## Item 1

### View C++ as a federation of languages
C++ consists of 4 primary sublanguages
- C
    * most features from **C** are available in C++
    * including blocks, statements, the preprocessor, built-in data types, arrays, pointers, etc.
- Object-Oriented C++
    * this part is about the **classes**
    * which are related to encapsulation, inheritance, dynamic binding, polymorphism, etc.
- Template C++
    * this part is about the **generic** programming
    * which is related to type-independent codes
- The STL
    * the STL is a template library which are used in various circumstances
- the point is that effective programming requires that you change strategy when you switch from one sublanguage to another
    * for instance, pass-by-value is generally more efficient than pass-by-reference for built-in types, 
    * but when you move from the C part of C++ to Object-Oriented C++, pass-by-reference-to-`const` is usually better


## Item 2

### Prefer `const`s, `enum`s, and `inline`s to `#define`s
Although `#include` and `#ifdef`/`#ifndef` are still useful
- for simple constants, prefer `const` objects or `enum`s to `#define`s
- for function-like macros, prefer `inline` functions to `#define`s


## Item 3

### Use `const` whenever possible
- if you're certain that the object doesn't need to be changed, try to make that object as `const`
    * so that you can get **help from the compiler** regarding error detection
- if non`const` and `const` member functions have the same definitions
    * make non`const` version call the `const` version through casting to avoid code duplication
        ```c++
        class ClassName {
        public:
            const char& operator[](const std::size_t position) const {
                // same definition
                return text[position];
            }
            char& operator[](const std::size_t position) {
                // same definition
                return const_cast<char&>(static_cast<const ClassName&>(*this)[position]);
            }
            // other members
        }; 
        ```


## Item 4

### Make sure that objects are initialized before they're used
Try to manually initialize objects of built-in types
- because they are usually **undefined** if the initiailizer is not provided

### `static` objects
There are 2 types of `static` objects
- local `static` objects
    * they are the `static` objects declared inside functions
- non-local `static` objects
    * other `static` objects
        * such as, global objects, objects defined at namespace scope, objects declared `static` inside classes, objects declared `static` at file scope
    * are non-local `static` objects

### A Translation Unit
A *translation unit* is the source code giving rise to a single object file
- it's basically a single source file, plus all of its `#include` files

### The Undefined Order of Initialization
It's worth noting that the relative order of initialization of non-local `static` objects defined in **different** translation units is **undefined**
- the solution to this problem is to move each non-local `static` objects into its own **functions**, where it's declared as `static`
    * that function returns the reference to that `static` object
- this solution is based on the C++'s guarantee that local `static` objects are initialized when the object's definition is **first encountered** during a **call** to that function
    ```c++
    class ClassName { /* definition */ };
    ClassName& getStaticData() {
        static ClassName data;
        return data;
    }
    ```

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}