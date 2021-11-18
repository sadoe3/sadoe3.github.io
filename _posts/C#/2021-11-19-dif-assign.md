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
- C++에서는 reference type으로 지정하지않으면, class의 instance를 포함한 모든 object는 assignment operator를 만났을때 copy를 하는것이 기본적인 처리방식이다.
- reference type으로 지정해야만, assignment operator를 만났을 때, 해당 object의 reference를 얻는 것임.
- 사실 reference라는 개념은 그냥 기존에 존재하던 object의 다른 이름이라고 생각하면 편함.
    - 즉 reference object는 별도의 메모리를 안에서 존재하지 않음.
    - 그냥 reference object가 만들어지면, 특정 object의 다른 이름이 만들어졌다고 생각하면 되고, 그 이름을 통해서도 기존에 존재하던 object를 access할 수 있게 한다라는 개념으로 생각하면 됨.
    - 즉, 그냥 다른 이름이기 때문에, reference object와 refernce object가 references to 하는 기존의 오브젝트는 서로 같은 object라고 생각하면 됨
- copy의 개념은 말 그대로 동일한 데이터를 갖는 object가 별도의 memory에 존재한다는 의미임.
    - 즉, 이 두개의 object는 서로 다른 object라고 생각해야함

## Assignment operator in C#
- C#에서는 refernce type의 모든 instance는 assignment operator를 만나면 기본적으로 해당 object의 reference를 얻는것이 기본적인 처리방식이다.
- reference type을 대상으로 copy를 하고 싶은 경우, 해당 object에서 `.Clone()` method를 call을 해서, 동일한 데이터를 갖고 있는 새로운 object에 대한 reference를 얻는 식으로 유사 copy를 하면 된다.
    - 이 방식은 마치 C에서 pointer를 통해 유사 call-by-reference를 구현한 것과 비슷한 개념이라고 생각하면 됨
- value type의 경우, 모든 instance는 assignment operator를 만나면 기본적으로 copy를 하는 것이 기본적인 처리방식이다.

## Differences
- 즉, C++은 class type의 instance던 int같은 built-in type의 instance던 assignment operator를 만났을때 reference로 받을지 copy로 받을지를 선택할 수 있음
- 그러나, C#은 class type의 instance는 무조건 reference로 받아야되고, 그렇지 않은 type의 instance는 무조건 copy로 받아야함
    - 그래서 c#에서 int같은 built-in type을 reference로 받기 위해선, 해당 type을 data member로 갖고 있는 class를 만들어서, 해당 class를 사용해야함 (이런 class를 wrapper class라고 함)
    - 또한 class를 copy로 받기 위해선, `clone`을 사용할 수도 있지만, 애초에 해당 class를 만들때 class로 만드는게 아니라 struct로 만들어서 사용하면 된다.
- 그래서 C++이나 C#은 모두 동일한 기능을 지원하긴 하지만, 이들을 수행하기 위해선 서로 다른 방법을 사용해야한다는 것을 알 수 있다.
    - 사실 이런 점은 여러개의 언어들을 사용하다보면 자연스럽게 겪는 현상이긴하다.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}