---
title: "Static classes in C#"

categories:
    - csharp

tags:
    - [Programming Language, C#, Static Class]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2021-10-07
---

## Static classes
- C#에서 static class는 static property, staic method 같은 static member들로만 구성된 class를 말한다.
- Static class의 특징은 intantiate을 못하는 class라는 점이다.
- 그렇기 때문에, static class를 사용하기 위해선, class의 이름을 이용하여 마치 class를 object처럼 사용하는 느낌으로 static class를 사용하면 된다.

```c#
ClassName.MethodName();     // static class usage example
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}