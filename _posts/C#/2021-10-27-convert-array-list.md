---
title: "How to convert an array to a list C#"

categories:
    - csharp

tags:
    - [Programming Language, C#, ToList]

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


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}