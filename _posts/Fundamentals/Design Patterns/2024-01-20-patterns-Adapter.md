---
title: "Design Patterns : Adapter"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Structural Pattern, Adapter]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-20
---

# Structural Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Adapter

### Also Known As
Wrapper


## Problem

### Intent
Convert the interface of a class into another interface clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.

### Applicability
Use the **Adapter** pattern when
- you want to use an existing class, and its interface does not match the one you need.
- you want to create a reusable class that cooperates with unrelated or unforeseen classes, that is, classes that don't necessarily have compatible interfaces.
- (*object adapter only*) you need to use several existing subclasses, but it's impractical to adapt their interface by subclassing every one. An object adapter can adapt the interface of its parent class.


## Solution

### Motivation
Suppose that ~
- The **problem** is that ~
- A **solution** is to ~

### Participants
- **Target** (`Shape`)
    * defines the domain-specific interface that Client uses.
- **Client** (`DrawingEditor`)
    * collaborates with objects conforming to the Target interface.
- **Adaptee** (`Text View`)
    * defines an existing interface that needs adapting.
- **Adapter** (`TextShape`)
    * adapts the interface of Adaptee to the Target interface.


### Sample Code
```c++

```

### Implementation
Although the implementation of Adapter is usually straightforward, here are some issues to keep in mind:
1. Implementing class adapters in C++. In a C++ implementation of a class adapter, Adapter would inherit publicly from Target and privately from Adaptee. Thus Adapter would be a subtype of Target but not of Adaptee.
2. Pluggable adapters. Let's look at three ways to implement pluggable adapters for the TreeDisplay widget described earlier, which can lay out and display a hierarchical structure automatically.
The first step, which is common to all three of the implementations discussed here, is to find a "narrow" interface for Adaptee, that is, the smallest subset of operations that lets us do the adaptation. A narrow interface consisting of only a couple of operations is easier to adapt than an interface with dozens of operations.
For TreeDisplay, the adaptee is any hierarchical structure. A minimalist interface might include two operations, one that defines how to present a node in the hierarchical structure graphically, and another that retrieves the node's children.
The narrow interface leads to three implementation approaches:
(a) Using abstract operations. Define corresponding abstract operations for the narrow Adaptee interface in the TreeDisplay class. Subclasses must implement the abstract operations and adapt the hierarchically structured object. For example, a Directory TreeDisplay subclass will implement these operations by accessing the directory structure.
Directory TreeDisplay specializes the narrow interface so that it can display directory structures made up of FileSystemEntity objects.
(b) Using delegate objects. In this approach, TreeDisplay forwards requests for accessing the hierarchical structure to a delegate object. TreeDisplay can use a different adaptation strategy by substituting a different delegate.
For example suppose there exists a DirectoryBrowser that uses a TreeDisplay.
DirectoryBrowser might make a good delegate for adapting
TreeDisplay to
the hierarchical directory structure. In dynamically typed languages like Smalltalk or Objective C, this approach only requires an interface for registering the delegate with the adapter. Then TreeDisplay simply forwards the requests to the delegate. NEXTSTEP [Add94] uses this approach heavily to reduce subclassing.
Statically typed languages like C++ require an explicit interface definition for the delegate. We can specify such an interface by putting the narrow interface that TreeDisplay requires into an abstract TreeAccessorDelegate class. Then we can mix this interface into the delegate of our choice-Directory Browser in this case-using inheritance. We use single inheritance if the DirectoryBrowser has no existing parent class, multiple inheritance if it does. Mixing classes together like this is easier than introducing a new TreeDisplay subclass and implementing its operations individually.
(c) Parameterized adapters. The usual way to support pluggable adapters in Smalltalk is to parameterize an adapter with one or more blocks. The block construct supports adaptation without subclassing. A block can adapt a request, and the adapter can store a block for each individual request. In our example, this means TreeDisplay stores one block for converting a node into a GraphicNode and another block for accessing a node's children.


### Related Patterns
- Bridge has a structure similar to an object adapter, but Bridge has a different intent: It is meant to separate an interface from its implementation so that they can be varied easily and independently An adapter is meant to change the interface of an existing object.
- Decorator enhances another object without changing its interface. A decorator is thus more transparent to the application than an adapter is. As a consequence, Decorator supports recursive composition, which isn't possible with pure adapters.
- Proxy defines a representative or surrogate for another object and does not change its interface.


## Consequences
Class and object adapters have different trade-offs. A class adapter
• adapts Adaptee to Target by committing to a concrete Adapter class. As a con-sequence, a class adapter won't work when we want to adapt a class and all its subclasses.
• lets Adapter override some of Adaptee's behavior, since Adapter is a subclass of Adaptee.
• introduces only one object, and no additional pointer indirection is needed to get to the adaptee.
An object adapter
• lets a single Adapter work with many Adaptees-that is, the Adaptee itself and all of its subclasses (if any). The Adapter can also add functionality to all Adaptees at once.
• makes it harder to override Adaptee behavior. It will require subclassing Adaptee and making Adapter refer to the subclass rather than the Adaptee itself.
Here are other issues to consider when using the Adapter pattern:
1. How much adapting does Adapter do? Adapters vary in the amount of work they do to adapt Adaptee to the Target interface. There is a spectrum of possible work, from simple interface conversion-for example, changing the names of opera-tions—to supporting an entirely different set of operations. The amount of work Adapter does depends on how similar the Target interface is to Adaptee's.
2. Pluggable adapters. A class is more reusable when you minimize the assumptions other classes must make to use it. By building interface adaptation into a class, you eliminate the assumption that other classes see the same interface. Put another way, interface adaptation lets us incorporate our class into existing systems that might expect different interfaces to the class. ObjectWorks \Smalltalk [Par90] uses the term pluggable adapter to describe classes with built-in interface adaptation.
Consider a TreeDisplay widget that can display tree structures graphically. If this were a special-purpose widget for use in just one application, then we might require the objects that it displays to have a specific interface; that is, all must descend from a Tree abstract class. But if we wanted to make TreeDisplay more reusable (say we wanted to make it part of a toolkit of useful widgets), then that requirement would be unreasonable. Applications will define their own classes for tree structures. They shouldn't be forced to use our Tree abstract class. Different tree structures will have different interfaces.
In a directory hierarchy, for example, children might be accessed with a Get-Subdirectories operation, whereas in an inheritance hierarchy, the corresponding operation might be called GetSubclasses. A reusable TreeDisplay widget must be able to display both kinds of hierarchies even if they use different interfaces. In other words, the TreeDisplay should have interface adaptation built into it.
We'll look at different ways to build interface adaptation into classes in the Implementation section.
3. Using two-way adapters to provide transparency. A potential problem with adapters is that they aren't transparent to all clients. An adapted object no longer conforms to the Adaptee interface, so it can't be used as is wherever an Adaptee object can. Two-way adapters can provide such transparency. Specifically, they're useful when two different clients need to view an object differently.


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}