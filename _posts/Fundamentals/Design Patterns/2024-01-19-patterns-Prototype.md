---
title: "Design Patterns : Prototype"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Creational Pattern, Prototype]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-19
---

# Creational Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Prototype


## Problem

### Intent
Specify the kinds of objects to create using a **prototypical** instance, and create new objects by **copying** this prototype.

### Applicability
Use the **Prototype** pattern when a system should be **independent** of how its products are created, composed, and represented; *and*
- when the classes to **instantiate** are specified at **run-time**
    * for example, by dynamic loading; *or*
- to **avoid** building a class **hierarchy** of factories that **parallels** the class **hierarchy** of products; *or*
- when instances of a class can have one of only a few different combinations of state.
    * It may be more convenient to install a corresponding number of prototypes and clone them rather than instantiating the class manually, each time with the appropriate state.


## Solution

### Motivation
Suppose that you want to implement multiple standards to draw objects in your application just like the example of the [**Abstract Factory**](https://sadoe3.github.io/design-patterns/patterns-AbstractFactory/) pattern. With the abstract factory pattern, you can consistenly draw objects of a **single standard**. But in this case, you want to draw objects of the **mixed standards**.
- The **problem** is that you have to **implement various derived classes** even if some of them draw **only one object differently**
- A **solution** is to utilize **composition** of the handles to the base classes of the objects to draw

### Participants
- **Prototype** (`WidgetPrototypeFactory`)
    * declares an interface for cloning itself.
- **ConcretePrototype** (`mixedWidgetFactory`, `reverselyMixedWidgetFactory`)
    * implements an operation for cloning itself.
- **Client** (`Application`)
    * creates a new object by asking a prototype to clone itself.

### Sample Code
```c++

```

### Implementation
Prototype is particularly useful with static languages like C++, where classes are not objects, and little or no type information is available at run-time. It's less important in languages like Smalltalk or Objective C that provide what amounts to a prototype (i.e., a class object) for creating instances of each class. This pattern is built into prototype-based languages like Self [US871, in which all object creation happens by cloning a prototype.
Consider the following issues when implementing prototypes:
1. Using a prototype manager. When the number of prototypes in a system isn't fixed (that is; they can be created and destroyed dynamically), keep a registry of available prototypes. Clients won't manage prototypes themselves but will store and retrieve them from the registry. A client will ask the registry for a prototype before cloning it. We call this registry a prototype manager.
A prototype manager is an associative store that returns the prototype matching a given key. It has operations for registering a prototype under a key and for unreg-istering it. Clients can change or even browse through the registry at run-time.
This lets clients extend and take inventory on the system without writing code.
2. Implementing the Clone operation. The hardest part of the Prototype pattern is implementing the Clone operation correctly. It's particularly tricky when object structures contain circular references.
Most languages provide some support for cloning objects. For example, Smalltalk provides an implementation of copy that's inherited by all subclasses of Object.
C++ provides a copy constructor. But these facilities don't solve the "shallow copy versus deep copy" problem [GR83]. That is, does cloning an object in turn clone its instance variables, or do the clone and original just share the variables?
A shallow copy is simple and often sufficient, and that's what Smalltalk provides by default. The default copy constructor in C++ does a memberwise copy, which means pointers will be shared between the copy and the original. But cloning prototypes with complex structures usually requires a deep copy, because the clone and the original must be independent. Therefore you must ensure that the clone's components are clones of the prototype's components. Cloning forces you to decide what if anything will be shared.
If objects in the system provide Save and Load operations, then you can use them to provide a default implementation of Clone simply by saving the object and loading it back immediately. The Save operation saves the object into a memory buffer, and Load creates a duplicate by reconstructing the object from the buffer.
3. Initializing clones. While some clients are perfectly happy with the clone as is, others will want to initialize some or all of its internal state to values of their choos-ing. You generally can't pass these values in the Clone operation, because their number will vary between classes of prototypes. Some prototypes might need multiple initialization parameters; others won't need any. Passing parameters in the Clone operation precludes a uniform cloning interface.
It might be the case that your prototype classes already define operations for (re)setting key pieces of state. If so, clients may use these operations immediately after cloning. If not, then you may have to introduce an Initialize operation (see the Sample Code section) that takes initialization parameters as arguments and sets the clone's internal state accordingly. Beware of deep-copying Clone operations-
—the copies may have to be deleted (either explicitly or within
Initialize) before you reinitialize them.

### Related Patterns
Prototype and Abstract Factory are competing patterns in some ways, as we discuss at the end of this chapter. They can also be used together, however. An Abstract Factory might store a set of prototypes from which to clone and return product objects.
Designs that make heavy use of the Composite and Decorater patterns often can benefit from Prototype as well.


## Consequences
Prototype has many of the same consequences that Abstract Factory and Builder have:
It hides the concrete product classes from the client, thereby reducing the number of names clients know about. Moreover, these patterns let a client work with application-specific classes without modification.
Additional benefits of the Prototype pattern are listed below.
1. Adding and removing products at run-time. Prototypes let you incorporate a new concrete product class into a system simply by registering a prototypical instance with the client. That's a bit more flexible than other creational patterns, because a client can install and remove prototypes at run-time.
2. Specifying new objects by varying values. Highly dynamic systems let you define new behavior through object composition-by specifying values for an object's variables, for example and not by defining new classes. You effectively define new kinds of objects by instantiating existing classes and registering the instances as prototypes of client objects. A client can exhibit new behavior by delegating responsibility to the prototype.
This kind of design lets users define new "classes" without programming. In fact, cloning a prototype is similar to instantiating a class. The Prototype pattern can greatly reduce the number of classes a system needs. In our music editor, one GraphicTool class can create a limitless variety of music objects.
3. Specifying new objects by varying structure. Many applications build objects from parts and subparts. Editors for circuit design, for example, build circuits out of subcircuits. For convenience, such applications often let you instantiate complex, user-defined structures, say, to use a specific subcircuit again and again.
The Prototype pattern supports this as well. We simply add this subcircuit as a prototype to the palette of available circuit elements. As long as the composite circuit object implements Clone as a deep copy, circuits with different structures can be prototypes.
4. Reduced subclassing. Factory Method often produces a hierarchy of Creator classes that parallels the product class hierarchy. The Prototype pattern lets you clone a prototype instead of asking a factory method to make a new object. Hence you don't need a Creator class hierarchy at all. This benefit applies primarily to languages like C++ that don't treat classes as first-class objects. Languages that do, like Smalltalk and Objective C, derive less benefit, since you can always use a class object as a creator. Class objects already act like prototypes in these languages.
5. Configuring an application with classes dynamically. Some run-time environments let you load classes into an application dynamically. The Prototype pattern is the key to exploiting such facilities in a language like C++.
An application that wants to create instances of a dynamically loaded class won't be able to reference its constructor statically. Instead, the run-time environment creates an instance of each class automatically when it's loaded, and it registers the instance with a prototype manager (see the Implementation section). Then the application can ask the prototype manager for instances of newly loaded classes, classes that weren't linked with the program originally. The ET++ application framework [WGM88] has a run-time system that uses this scheme.
The main liability of the Prototype pattern is that each subclass of Prototype must implement the Clone operation, which may be difficult. For example, adding Clone is difficult when the classes under consideration already exist. Implementing Clone can be difficult when their internals include objects that don't support copying or have circular references.


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}