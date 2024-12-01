---
title: "Assignment operator in C++ and C#"

categories:
    - csharp

tags:
    - [Programming Language, C++, C#, assignment, operator]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2021-11-19
---

## Assignment operator in C++
- In C++, unless specified as a reference type, all objects, including instances of classes, perform a copy operation when the assignment operator is used by default.
- Only if the object is specified as a reference type does the assignment operator assign the reference to the object.
- In fact, the concept of a reference can be thought of as just another name for an already existing object.
    - That is, a reference object does not exist in memory as a separate entity.
    - When a reference object is created, it’s simply another name for an existing object, and through that name, you can access the original object.
    - In other words, since it's just another name, the reference object and the original object it refers to are considered to be the same object.
- The concept of copying means that an object with the same data as the original is created in a separate memory location.
    - In this case, these two objects are considered distinct from each other.

## Assignment operator in C#
- In C#, every instance of a reference type, by default, receives the reference to the object when the assignment operator is used.
- If you want to copy a reference type object, you must call the `.Clone()` method on the object to obtain a reference to a new object with the same data.
    - This approach is similar to how C uses pointers for a call-by-reference-like behavior.
- For value types, every instance, by default, performs a copy when the assignment operator is used.

## Differences
- In C++, whether it is an instance of a class type or a built-in type like `int`, the assignment operator allows you to choose whether to receive a reference or a copy.
- However, in C#, an instance of a class type is always received by reference, while non-class types (value types) are always received by copy.
    - To receive a built-in type like `int` by reference in C#, you would need to create a class containing that type as a data member (known as a wrapper class).
    - To receive a class type by copy, you can use `Clone()`, but another option is to define the class as a `struct` instead of a `class` when it is initially created.
- Thus, while both C++ and C# provide the same functionality, they require different approaches to achieve these behaviors.
    - This is a common experience when working with multiple programming languages.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}