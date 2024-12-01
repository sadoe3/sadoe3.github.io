---
title: "How to convert an array to a list and create an array dynamically in C#"

categories:
    - csharp

tags:
    - [Programming Language, C#, ToList, Array, List]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2021-10-27
---

## How to convert an array to a list in C#
- There are various methods, but the one I will use is the `ToList()` method.
- To use this method, you need to include the `System.Collections.Generic` and `System.Linq` namespaces.
    ```c#
    // example of using ToList()
    int[] tempArr = { 1, 2, 3 };
    List<int> tempList = tempArr.ToList();
    ```

## Creating an array dynamically
- In C/C++, arrays are value types, so the size must be provided before compilation.
    - This means that dynamic arrays cannot be created.
        ```c
        // example in C
        int tempArry[3];
        ``` 
- However, in Java and C#, arrays are reference types, so the size can be provided at runtime.
    - This means that arrays can be created dynamically.
        ```c#
        // example of Unity script using C#
        int[] tempArry = FindObjectsOfType<BoxCollider2D>();
        ```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}