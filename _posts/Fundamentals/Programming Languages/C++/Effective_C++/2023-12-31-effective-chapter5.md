---
title: "Effective C++ : Chapter 5"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective C++, Class, Implementation]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-31
---

# Implementations

> 이 포스트는 Effective C++(3rd Edition)를 바탕으로 작성되었습니다.

## Item 26

### Postpone variable definitions as long as possible
```c++
// premature code
ClassName a;
if(/*some condition*/) {
    throw std::exception();
}
// a is never used when exception is thrown although it is constructed (unnecessary construction and destruction)


// refined code
if(/*some condition*/) {
    throw std::exception();
}
ClassName a;
// a is always used if it's constructed
```
- moreover, generally, try to define objects **before loop** to avoid unnecessary constructions and destructions


## Item 27

### Minimize casting
If a design requires casting, try to develop a cast-free alternative
- When the casting is necessary, try to hide it inside a function
    * clients can then call the function without having to worry about the casting issue
- prefer C++ style casts to old-style casts
    * they are easier to see, and they are more specific about what they do


## Item 28

### Avoid returning "handles" to object internals
Minimize the number of methods that return handles (pointers or references) to data members
- because it's **inversely proportional** to the **encapsulatedness**
- also it helps `cosnt` member functions act `const` and minimize the creation of dangling handles
    * dangling hanldes is the handles that point to the objects which don't exist


## Item 29

### Strive for exception-safe code
Exception-safe code must offer one of the 3 guarantees
- the **basic** guarantee
    * it's acquired by promising that there's **no memory leak** and **no data corruption**
    * with this level of guarantee, we can't exactly predict whether the current state of the object is the default one or previous one
- the **strong** guarantee
    * it's acquired by promising that the **basic** guarantee is acquired
    * and after handling the exception, the state is always the previous one so that the program state is as if they were never been called
        + this can be easily implemented by copy-and-swap technique
        + which is use the local variable so that the corruption or modification on that object doesn't matter
- the **nothrow** guarantee
    * it's acquired by promising that the function doesn't throw an exeption
    * it can be implemented by using `noexcept` keyword
    * if that function is trying to throw an exception, then an error occurs
- a function can usually offer a guarantee no stronger than the weakest guarantee of the functions it calls


## Item 30

### Understand the ins and outs of inlining
Try to limit most inlining to **small**, **frequently called** functions in order to increase performance
- otherwise, performance may decrease
- moreover, try to put `inline` functions inside header files
    * because inlining usually happens at compile time
    * but this **doesn't mean** that inlining every fuction template just because they appear in header files is right


## Item 31

### Minimize compilation dependencies between files
If the compilation dependency is high, the compiler would recompile everything although you made a minor change to the implementation of a class 
- in order to avoid this issue, try to make the code depend on the **declarations** not the definitions
    * we can do so by using the handlers (pointers or references) instead of acutal objects
        + because for handlers only declaration is needed
        + however, actual objects need the definition to be constructed
    * the textbook introduced handle class and interface class as examples
- providing 2 header files for delcarations-only and full contents is recommended for library authors 

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}