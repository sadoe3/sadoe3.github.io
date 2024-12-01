---
title: "Similarity and difference between #include in C and using in C#"

categories:
    - csharp

tags:
    - [Programming Language, C#, C]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2021-10-06
---

## Similarity
- Both allow the use of code that is not defined in the source file being written.

## Difference
- A key difference occurs after compilation.
- In C or C++, when using `#include`, the corresponding header file is included in the source file.
- This means that by using `#include`, the header file is integrated into the source file, which effectively changes the source code.
- However, in C#, simply accessing the relevant namespace does not modify the source code, even after compilation.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}