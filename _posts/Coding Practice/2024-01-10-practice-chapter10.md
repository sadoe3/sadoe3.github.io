---
title: "Clean Code : Chapter 10"

categories:
    - coding-practice

tags:
    - [Clean Code, Coding Practice, Class]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-10
---

# Classes	

> 이 포스트는 Clean Code(1st Edition)를 바탕으로 작성되었습니다.

## The Single Responsibility Principle 

### Single Responsibility
A **class** should have **one, and only one**, *reason to change*
- if a class contains **multiple** responsibilities, you need to break it into **decoupled** classes with **single** responsibilities
    * then, there would be **many small** classes
    * and, a system with **many small** classes is far **better** than a system with **a few large** classes

### How to Achieve
You can achieve **SRP** by following the steps below
1. implement a **"God"** class which does what you want
2. analyze its **responsibilities**
3. implement **multiple small classes** which **respectively** have **single** responsibility among the ones which the **"God"** class has
4. implement one or more **high-level** classes which **use** the **small** classes through **composition** or **inheritance**
    - this mechanism is similar to the one from the [**functions**](https://sadoe3.github.io/coding-practice/practice-chapter3/#one-function-one-purpose) where a **high-level** purpose can be achieved through achieving one or more **intermediate-level** purposes


cohesion
cohesion = how many data members are used by the each methods -> maximally cohesive = a class in which each variable is used by each method -> maximally cohesive is not advisable and impossible -> high cohesion is recommended -> why? -> low cohesion means that the number of data members which are used by a certain number of methods is high -> when this happens, it almost alwyas means that you should try to separate the data members and methods into two or more classes such that the new classes are move cohesive -> ultimately, classes would have a small number of data members


flexible to changes
for most systems, change is continual. hence we need to implement a class which reduces the risk of the change
we want to structure our systems so that we edit as little as possible when we update them with new or changed features
in an ideal system, we incorporate new features by extending the system, not by making modifications to existing code 
one of ways to achieve this objective is to minimize the dependencies between classes
-> exmaple: bad -> class depend upon the implementation of another class -> is at risk when those implementations change -> clean -> class depend upon the interface -> is can be done in c++ by using abstract base class(primer link) -> by minimizing coulging in this way ~ (p.150)




[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}