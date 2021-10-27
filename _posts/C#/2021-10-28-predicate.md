---
title: "Predicate in C#"

categories:
    - csharp

tags:
    - [Programming Language, C#, predicate]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2021-10-28
---

## Predicate
- Predicate이란 입력값이 없거나 하나이고, return 값이 bool type인 delegate이다.
- Predicate은 lambda expression으로 표현되며, 보통 generic collection과 함께 쓰인다.
- 사용방법은 predicate을 argument로 보내야 할때, 다음과 같이 사용하면 된다
```c#
// example of using predicate in C#
List<string> tempList = new List<string> { "A", "B", "C" };
Console("Find A: " + tempList.Find(x => x == "A"));
// x is an element of the given collection object
// the name of the element doesn't have to be x, you can name it freely like s or so -> ex: s => s == "A"
// x == "A" is the condition
```
- 다음은 Unity script에서 입력값이 없이 predicate을 사용하는 예제이다.
```c#
while(.../* some condition */) {
    ... // some codes
    yield return new WaitUntil(() => tempList.Count > 0);
    // if you don't want to pass any argument, then you need to type empty parenthesis () instead of the name of the element
}
```

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}