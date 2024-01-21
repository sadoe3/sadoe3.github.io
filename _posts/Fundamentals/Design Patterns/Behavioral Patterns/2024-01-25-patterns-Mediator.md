---
title: "Design Patterns : Mediator"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, Mediator]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-25
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Mediator


## Problem

### Intent
Define an object that encapsulates how a set of objects interact. Mediator promotes loose coupling by keeping objects from referring to each other explicitly, and it lets you vary their interaction independently.

### Applicability
Use the **Mediator** pattern when
- a set of objects communicate in well-defined but complex ways.
    * The resulting interdependencies are unstructured and difficult to understand.
- reusing an object is difficult because it refers to and communicates with many other objects.
- a behavior that's distributed between several classes should be customizable without a lot of subclassing.


## Solution

### Motivation
Suppose that ~
- The **problem** is that ~
- A **solution** is to ~

### Participants
- **Mediator** (`DialogDirector`)
    * defines an interface for communicating with Colleague objects.
- **ConcreteMediator** (`FontDialogDirector`)
    * implements cooperative behavior by coordinating Colleague objects.
    * knows and maintains its colleagues.
- **Colleague classes** (`ListBox`, `EntryField`)
    * each Colleague class knows its Mediator object.
    * each colleague communicates with its mediator whenever it would have other wise communicated with another colleague.

### Sample Code
```c++

```

### Implementation
The following implementation issues are relevant to the Mediator pattern:
1. *Omitting the abstract Mediator class*
    * There's no need to define an abstract Mediator class when colleagues work with only one mediator. The abstract coupling that the Mediator class provides lets colleagues work with different Mediator sub-classes, and vice versa.
2. *Colleague-Mediator communication*
    * Colleagues have to communicate with their mediator when an event of interest occurs. One approach is to implement the Mediator as an Observer using the Observer pattern. Colleague classes act as Subjects, sending notifications to the mediator whenever they change state. The mediator responds by propagating the effects of the change to other colleagues.
    * Another approach defines a specialized notification interface in Mediator that lets colleagues be more direct in their communication. Smalltalk/V for Windows uses a form of delegation: When communicating with the mediator, a colleague passes itself as an argument, allowing the mediator to identify the sender. The Sample Code uses this approach, and the Smalltalk/V implementation is discussed further in the Known Uses.

### Related Patterns
- **Facade** differs from Mediator in that it abstracts a subsystem of objects to provide a more convenient interface. Its protocol is unidirectional
    * that is, Facade objects make requests of the subsystem classes but not vice versa.
    * In contrast, Mediator enables cooperative behavior that colleague objects don't or can't provide, and the protocol is multidirectional.
* Colleagues can communicate with the mediator using the **Observer** pattern.


## Consequences
The Mediator pattern has the following benefits and drawbacks:
1. *It limits subclassing*
    * A mediator localizes behavior that otherwise would be distributed among several objects. Changing this behavior requires subclassing Mediator only; Colleague classes can be reused as is.
2. *It decouples colleagues*
    * A mediator promotes loose coupling between colleagues.
You can vary and reuse Colleague and Mediator classes independently.
3. *It simplifies object protocols*
    * A mediator replaces many-to-many interactions with one-to-many interactions between the mediator and its colleagues. One-to-many relationships are easier to understand, maintain, and extend.
4. *It abstracts how objects cooperate*
    * Making mediation an independent concept and encapsulating it in an object lets you focus on how objects interact apart from their individual behavior. That can help clarify how objects interact in a system.
5. *It centralizes control*
    * The Mediator pattern trades complexity of interaction for complexity in the mediator. Because a mediator encapsulates protocols, it can become more complex than any individual colleague. This can make the mediator itself a monolith that's hard to maintain.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}