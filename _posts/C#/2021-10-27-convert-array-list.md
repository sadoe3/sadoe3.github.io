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

## How to convert an array to a list C#
- 다양한 방법이 있지만, 내가 사용할 방법은 ToList()를 사용하는 방법이다.
- 이 방법을 쓰기 위해선, System.Collections.Generic과 System.Linq namespace를 using 해줘야 한다.
```c#
// example of using ToList()
int[] tempArr = { 1, 2, 3 };
List<int> tempList = tempArr.ToList();
```

## Creating an array dynamically
- C/C++에서 array는 value type이기 때문에, compile이 되기 전에 미리 size를 제공해줘야한다.
    - 즉, 동적으로 array를 만들수가 없다.
    ```c
    // example in C
    int tempArry[3];
    ``` 
- 그러나 Java나 C#에서는 array가 reference type이기 때문에, runtime에서 size를 제공해줘도 된다.
    - 즉, 동적으로 array를 만들 수 있다.
    ```c#
    // example of Unity script using C#
    int[] tempArry = FindObjectsOfType<BoxCollider2D>();
    ```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}