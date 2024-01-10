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


## Cohesion

### Maximally Cohesive Classes
A class in which **each** variable is used by **each** method is **maximally cohesive**
- in general it is **neither** advisable nor possible to create such **maximally cohesive** classes; on the other hand, **high cohesive** classes are **preferred**

### High Cohesive Classes
In general the **more variables** a **method** manipulates the **more cohesive** that method is to its class
- if cohesion is **low**, it means that the **number** of data members which are **used** by **a certain number** of methods is **high** 
    * when this happens, it almost always means that you should try to **separate** the data members and the methods into two or more classes such that the **new classes** are **move cohesive**
- ultimately, **high cohesive** classes would have **a small number** of data members


## Flexibility to Changes

### Change is Continual
For most systems, **needs** will change, therefore code will **change** 
- hence we need to implement a class which **reduces the risk** of the change

### Ideal Classes
We want to structure our systems so that we **edit as little as possible** when we **update** them with new or changed features
- in an ideal system, we incorporate **new features** by **extending** the system, **not by making modifications** to existing code 

### How to Achieve
One of the ways to implement such **flexible** classes is to **minimize** the **dependencies** between classes
- if a client class **depends** upon the **implementation** of another class
    * it would be at **risk** when those implementations **change**
- hence, we can **clean** this class by making it **depending** upon the **interface** of another class
    * this can be done by using [**pure virtual**](https://sadoe3.github.io/cpp/primer-chapter15/#abstract-base-classes) methods in C++
- by **minimizing coupling** in this way, our classes adhere to another class design principle known as the **Dependency Inversion Principle** (DIP)
    * in essence, the **DIP** says that our classes should **depend** upon **abstractions**, not on concrete details


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}