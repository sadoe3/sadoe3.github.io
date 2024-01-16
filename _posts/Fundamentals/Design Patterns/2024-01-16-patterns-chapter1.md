---
title: "Design Patterns : Chapter 1"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-16
---

# Introduction

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Reusable Object-Oriented Design
When designers find a **good solution**, they use it **again and again**.
- such experience is part of what makes them **experts**.
- this book **records** those good solutions so that you can use them next time **more easily**


## What Is a Design Pattern?

### 4 Essential Elements
In general, a **pattern** has 4 essential elements
1. The **pattern name** is a handle we can use to describe a design problem, its solutions, and consequences in a word or two.
    * Naming a pattern immediately increases our design vocabulary.
    * It lets us design at a higher level of abstraction.
    * Having a vocabulary for patterns lets us talk about them with our colleagues, in our documentation, and even to ourselves.
    * It makes it easier to think about designs and to communicate them and their trade-offs to others.
2. The **problem** describes when to apply the pattern.
    * It explains the problem and its context.
    * It might describe specific design problems such as how to represent algorithms as objects.
    * It might describe class or object structures that are symptomatic of an inflexible design.
    * Sometimes the problem will include a list of conditions that must be met before it makes sense to apply the pattern.
3. The **solution** describes the elements that make up the design, their relationships, responsibilities, and collaborations.
    * The solution doesn't describe a particular concrete design or implementation, 
        + because a pattern is like a template that can be applied in many different situations.
    * Instead, the pattern provides an abstract description of a design problem and how a general arrangement of elements (classes and objects in our case) solves it.
4. The **consequences** are the results and trade-offs of applying the pattern.
    - Though consequences are often unvoiced when we describe design decisions
        * they are critical for evaluating design alternatives and for understanding the costs and benefits of applying the pattern.


## 1.5 Organizing the Catalog

### Organized by the Purposes
**Patterns** can be organized by their **purposes** which reflect what they do
- Patterns can have either creational, structural, or behavioral purpose. 
    * **Creational** patterns concern the process of object creation.
    * **Structural** patterns deal with the composition of classes or objects.
    * **Behavioral** patterns characterize the ways in which classes or objects interact and distribute responsibility.

### Other Ways to Organize the Patterns
- There are **other ways** to organize the patterns.
    * Some patterns are often used **together**.
        + For example, Composite is often used with Iterator or Visitor.
    * Some patterns are **alternatives**:
        + Prototype is often an alternative to Abstract Factory.
    * Some patterns result in **similar designs** even though the patterns have **different intents**.
        + For example, the structure diagrams of Composite and Decorator are similar.
- Yet another way to organize design patterns is according to how they reference each other in their "**Related Patterns**" sections.
    * Having multiple ways of thinking about patterns will deepen your insight into what they do, how they compare, and when to apply them.


## How Design Patterns Solve Design Problems

### Program to an Interface, Not an Implemenation 
There are two benefits to manipulating objects **solely in terms of the interface** defined by abstract classes:
1. Clients remain unaware of the specific types of objects they use, as long as the objects adhere to the interface that clients expect.
2. Clients remain unaware of the classes that implement these objects.
    * Clients only know about the abstract class(es) defining the interface.
- this reduces **implementation dependencies** between subsystems so that you get **high flexibility** regarding the implementation of the derived classes

### Putting Reuse Mechanisms to Work
The two most common techniques for resuing functionality in object-oriented systems are **class inheritance** and **object composition**
- class inheritance lets you define the implementation of one class interms of another's
    * reuse by **subclassing** is of ten referred to as **white-box reuse**
        + the term "white-box" refers to visibility
        + with inheritance, the internals of parent classes are often visible to subclasses
- on the other hand, new functionality can be obtained by **composing** objects to get more complex functionality
    * reuse by **composing** is called **black-box reuse**
        + because no internal details of objects are visible
- inheritance and composition each have their advantages and disadvantages
    * however, by default, **favor** object **composition** over class inheritance
        + because designs are often made **reusable** (and **simpler**) by depending more on object **composition**

### Common Causes of Redesign
Here are some common **causes of redesign** along with the design pattern(s) that address them:
1. Creating an object by specifying a class explicitly
    * Specifying a class name when you create an object commits you to a particular implementation instead of a particular interface. This commitment can complicate future changes. To avoid it, create objects indirectly.
    * Design patterns: Abstract Factory, Factory Method, Prototype.
2. Dependence on specific operations
    * When you specify a particular operation, you commit to one way of satisfying a request. By avoiding hard-coded requests, you make it easier to change the way a request gets satisfied both at compile-time and at run-time.
    * Design patterns: Chain of Responsibility, Command.
3. Dependence on hardware and software platform
    * External operating system interfaces and application programming interfaces (APIs) are different on different hardware and software platforms. Software that depends on a particular platform will be harder to port to other platforms. It may even be difficult to keep it up to date on its native platform. It's important therefore to design your system to limit its platform dependencies.
    * Design patterns: Abstract Factory, Bridge.
4. Dependence on object representations or implementations
    * Clients that know how an object is represented, stored, located, or implemented might need to be changed when the object changes. Hiding this information from clients keeps changes from cascading.
    * Design patterns: Abstract Factory, Bridge, Memento, Proxy.
5. Algorithmic dependencies
    * Algorithms are often extended, optimized, and replaced during development and reuse. Objects that depend on an algorithm will have to change when the algorithm changes. Therefore algorithms that are likely to change should be isolated.
    * Design patterns: Builder, Iterator, Strategy, Template Method, Visitor.
6. Tight coupling
    * Classes that are tightly coupled are hard to reuse in isolation, since they depend on each other. Tight coupling leads to monolithic systems, where you can't change or remove a class without understanding and changing many other classes. The system becomes a dense mass that's hard to learn, port, and maintain.
    * Loose coupling increases the probability that a class can be reused by itself and that a system can be learned, ported, modified, and extended more easily. Design patterns use techniques such as abstract coupling and layering to promote loosely coupled systems.
    * Design patterns: Abstract Factory, Bridge, Chain of Responsibility, Command, Facade, Mediator, Observer.
7. Extending functionality by subclassing
    * Customizing an object by subclassing often isn't easy. Every new class has a fixed implementation overhead (initialization, finalization, etc.). Defining a subclass also requires an in-depth understanding of the parent class. For example, overriding one operation might require overriding another. An overridden operation might be required to call an inherited operation. And subclassing can lead to an explosion of classes, because you might have to introduce many new subclasses for even a simple extension.
    * Object composition in general and delegation in particular provide flexible alternatives to inheritance for combining behavior. New functionality can be added to an application by composing existing objects in new ways rather than by defining new subclasses of existing classes. On the other hand, heavy use of object composition can make designs harder to understand. Many design patterns produce designs in which you can introduce customized functionality just by defining one subclass and composing its instances with existing ones.
    * Design patterns: Bridge, Chain of Responsibility, Composite, Decorator, Observer, Strategy.
8. Inability to alter classes conveniently
    * Sometimes you have to modify a class that can't be modified conveniently. Perhaps you need the source code and don't have it (as may be the case with a commercial class library). Or maybe any change would require modifying lots of existing subclasses. Design patterns offer ways to modify classes in such circumstances.
    * Design patterns: Adapter, Decorator, Visitor.
- These examples reflect the flexibility that design patterns can help you build into your software.

### Toolkits and Frameworks
- Often an application will incorporate classes from one or more **libraries** of predefined classes called **toolkits**.
    * A toolkit is a set of **related** and **reusable** classes designed to provide useful, **general-purpose** functionality
    * Toolkits emphasize **code reuse**
        + When you use a toolkit (or a conventional subroutine library for that matter), you **write** the **main body** of the application and **call** the **code** you want to reuse.
- A **framework** is a set of cooperating classes that make up a reusable **design** for a specific class of **software**
    * Frameworks emphasize **design reuse**
        + When you use a framework, you **reuse** the **main body** and **write** the **code** it calls.
    * With frameworks, You'll have to write operations with particular names and calling conventions, but that **reduces** the **design decisions** you have to make.
    *  Not only can you **build** applications **faster** as a result, but the applications have **similar** structures.
        + They are **easier to maintain**, and they seem more consistent to their users.
        + On the other hand, you **lose some creative freedom**, since many design decisions have been made for you.


## How to Select a Design Pattern
Here are several different approaches to finding the design pattern that's right for your problem:
- Consider how design patterns solve design problems
- Scan **Intent** sections
    * Read through each pattern's intent to find one or more that sound relevant to your problem.
- Study how patterns interrelate
- Study patterns of like purpose.
    * The catalog has three chapters, one for creational patterns, another for structural patterns, and a third for behavioral patterns.
    * Each chapter starts off with introductory comments on the patterns and concludes with a section that compares and contrasts them.
    * These sections give you insight into the similarities and differences between patterns of like purpose.
- Examine a cause of redesign.
    * Look at the [**causes of redesign**]() to see if your problem involves one or more of them. Then look at the patterns that help you avoid the causes of redesign.
• Consider what should be variable in your design. This approach is the opposite of focusing on the causes of redesign. Instead of considering what might force a change to a design, consider what you want to be able to change without redesign. The focus here is on encapsulating the concept that varies, a theme of many design patterns. Table 1.2 lists the design aspects) that design patterns let you vary independently, thereby letting you change them without redesign.


## 1.8 p29


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}