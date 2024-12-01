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
- C# is an object-oriented language, and like other object-oriented languages, it supports Polymorphism.
- Polymorphism can be simply explained as follows:
    - By using the `override` feature in object-oriented languages, a method can accept a handle (e.g., pointer or reference) of a base class as a parameter. When calling the virtual method of the base class through this handle, depending on which derived class object is passed as the parameter, the unique definition of that object is applied, leading to different results each time. This phenomenon, where the same method call produces different, or varied, outcomes, is referred to as polymorphism.
- To implement polymorphism, the concept of Inheritance must be used.
- Inheritance is simply applied as follows:
    ```c#
    // example of inheritance in c#
    public class BaseClass {
        ... // codes
    }
    public class DerivedClass : BaseClass {
        ... // codes
    }
    ```
- When working with inheritance, there may be situations where the constructor of the base class needs to be called from the derived class. This can be implemented as follows:
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
- In C#, polymorphism through overriding can be implemented in the following way:
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
    ```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}