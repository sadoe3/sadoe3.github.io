---
title: "Polymorphism in C#"

categories:
    - csharp

tags:
    - [Programming Language, C#, Polymorphism]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2021-10-18
---

## Polymorphism
- C#은 객체지향언어이기 때문에, 다른 객체지향언어들처럼 Polymorphism(다형성)을 지원한다. 
- 다형성은 쉽게 설명하자면 다음과 같다.
    - 객체지향언어의 override기능을 이용하면, 특정 method에서 base class의 handle(ex- pointer, reference)를 parameter로 받아, 해당 handle의 virtual method를 call을 하게되면, 어떤 derived class의 오브젝트를 parameter로 받았는지에 따라, 해당 object의 고유한 definition이 적용되어, 매번 다른 결과가 나오게 된다. 이러한 현상을 동일한 method를 call을 해도 다른, 즉, 다양한 결과가 나온다라고 해석하여 다형성이라고 부른다.
- 다형성을 구현하기 위해선 Inheritance(상속)의 기능을 사용해야한다.
- 상속은 간단하게 다음과 같이 진행해주면 된다.
```c#
// example of inheritance in c#
public class BaseClass {
    ... // codes
}
public class DerivedClass : BaseClass {
    ... // codes
}
```
- 상속을 하다보면, base class의 constructor를 derived class에서 call을 해야하는 경우가 생기는데 다음과 같이 구현할 수 있다.
```c#
// example of calling base class's constructor in c#
public class BaseClass {
    public BaseClass(int tempInteger)
    ... // codes
}
public class DerivedClass : BaseClass {
    public DerivedClass(string tempString, int tempInteger) : base(tempInteger) 
    { 
        ... // codes
    }
    ... // codes
}
```
- C#에서 overriding을 통한 polymorphism은 다음과 같은 방법으로 구현할 수 있다.
```c#
// example of overriding in c#
public class BaseClass {
    ... // codes
    public virtual void methodName(int tempInt){
        ... // code A
    }
}
public class DerivedClass : BaseClass {
    ... // codes
    public override void methodName(int tempInt){
        ... // code B
    }
}
}
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}