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
- 둘다 작성하고 있는 Source file에 정의 되어 있지 않은 코드들을 사용할 수 있게 해준다.

## Difference
- 컴파일을 하고 난 뒤, 결정적인 차이점이 발생하게 된다.
- C나 C++의 #include의 경우, 해당 header file을 source file에 include 시킨다.
- 즉, #include를 이용하면, 해당 header file이 source file에 들어가기 때문에, source code가 변경된다는 것이다.
- 그러나 C#의 경우, 단순히 해당 namespace에 접근한다라는 기능만 수행하기 때문에, 컴파일을 마치고 나서도 source code가 변경되지 않는다는 것이다.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}