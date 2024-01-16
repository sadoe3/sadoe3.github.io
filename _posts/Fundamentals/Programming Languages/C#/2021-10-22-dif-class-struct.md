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

## differences in C/C++
- C에서는 struct를 structure 만들때 사용하고, class라는 keyword가 없다.
- C++에서는 둘다 class를 만들때 사용하며, 차이는 default access가 public이냐 private이냐의 차이만 존재한다.
    - class가 private, struct가 public

## Similarity
- C#에서 class와 struct는 모두 property와 method를 갖을 수 있다. 
- 그래서 대부분에 상황에서는 둘다 같은 사용법으로 코딩을 진행해도, 크게 문제가 발생되지 않는다.

## Difference
- 그러나 C#에서 class와 struct는 결정적인 차이를 갖게 된다.
- 바로 class는 reference type으로 instance가 만들어지고, struct는 value type으로 instance가 만들어진다는 것이다.
- 그래서 객체지향 자료형을 만들고 싶은데, reference type이 필요하면 class로 만들고, value type이 필요하면 struct로 만들어주면 된다.

## out keyword
- 위에서 언급한 차이가 있긴 해도, C#에는 out이라는 keyword를 이용하여 struct instance를 passed-by-reference의 형태로 method의 argument에 보낼 수 있게 해준다.
- 사용방법은 out keyword를 해당 argument 앞에 써주면 된다.
```c#
// example of using out keyword
tempFunction(out argumentName);
```

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}