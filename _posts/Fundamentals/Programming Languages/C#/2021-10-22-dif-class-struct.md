---
title: "Difference between classes and structures in C#"

categories:
    - csharp

tags:
    - [Programming Language, C#, class, struct]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2021-10-22
---

## Differences in C/C++
- In C, `struct` is used to define structures, and there is no `class` keyword.
- In C++, both `struct` and `class` can be used to define types, with the only difference being the default access level: `class` defaults to private, while `struct` defaults to public.

## Similarity
- In C#, both `class` and `struct` can have properties and methods. 
- Therefore, in most cases, you can use either one without major issues, as they share similar functionality.

## Difference
- However, in C#, `class` and `struct` have a crucial distinction.
- A `class` is a reference type, meaning instances are created as references, while a `struct` is a value type, meaning instances are created with their actual values.
- If you want to create an object-oriented type that requires a reference, use a `class`; if you need a value type, use a `struct`.

## `out` Keyword
- Despite the difference mentioned above, C# provides the `out` keyword, allowing you to pass a `struct` instance by reference to a method.
- To do this, simply place the `out` keyword before the argument in the method signature.
```c#
// example of using out keyword
tempFunction(out argumentName);
```

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}