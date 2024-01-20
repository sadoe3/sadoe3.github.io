---
title: "Design Patterns : Command"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, Command]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-24
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Command

### Also Known As
Action, Transaction


## Problem

### Intent
Encapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.

### Applicability
Use the **Command** pattern when you want to
- parameterize objects by an action to perform.
    * You can express such parameterization in a procedural language with a callback function
        + that is, a function that's registered somewhere to be called at a later point.
    * Commands are an object-oriented replacement for callbacks.
- specify, queue, and execute requests at different times.
    * A Command object can have a lifetime independent of the original request.
    * If the receiver of a request can be represented in an address space-independent way
        + then you can transfer a command object for the request to a different process and fulfill the request there.
- support undo
    * The Command's Execute operation can store state for reversing its effects in the command itself.
    * The Command interface must have an added Unexecute operation that reverses the effects of a previous call to Execute.
    * Executed commands are stored in a history list.
    * Unlimited-level undo and redo is achieved by traversing this list backwards and forwards calling Unexecute and Execute, respectively.
- support logging changes so that they can be reapplied in case of a system crash
    * By augmenting the Command interface with load and store operations
        + you can keep a persistent log of changes.
    * Recovering from a crash involves reloading logged commands from disk and reexecuting them with the Execute operation.


## Solution

### Motivation
Suppose that ~
- The **problem** is that ~
- A **solution** is to ~

### Participants
- **Command**
    * declares an interface for executing an operation.
- **ConcreteCommand** (`PasteCommand`, `OpenCommand`)
    * defines a binding between a Receiver object and an action.
    * implements Execute by invoking the corresponding operation(s) on Receiver.
- **Client** (`Application`)
    * creates a ConcreteCommand object and sets its receiver.
- **Invoker** (`Menultem`)
    * asks the command to carry out the request.
- **Receiver** (`Document`, `Application`)
    * knows how to perform the operations associated with carrying out a request.
    * Any class may serve as a Receiver.

### Sample Code
```c++

```

### Implementation
Consider the following issues when implementing the Command pattern:
1. *How intelligent should a command be?*
    * A command can have a wide range of abilities. At one extreme it merely defines a binding between a receiver and the actions that carry out the request. At the other extreme it implements everything itself without delegating to a receiver at all. The latter extreme is useful when you want to define commands that are independent of existing classes, when no suitable receiver exists, or when a command knows its receiver implicitly. For example, a command that creates another application window may be just as capable of creating the window as any other object. Somewhere in between these extremes are commands that have enough knowledge to find their receiver dynamically.
2. *Supporting undo and redo*
    * Commands can support undo and redo capabilities if they provide a way, to reverse their execution (e.g., an Unexecute or Undo opera-tion). A ConcreteCommand class might need to store additional state to do so.
    * This state can include
        + the Receiver object, which actually carries out operations in response to the request,
        + the arguments to the operation performed on the receiver, and
        + any original values in the receiver that can change as a result of handling the request. The receiver must provide operations that let the command return the receiver to its prior state.
    * To support one level of undo, an application needs to store only the command that was executed last. For multiple-level undo and redo, the application needs a **history list** of commands that have been executed, where the maximum length of the list determines the number of undo/redo levels. The history list stores sequences of commands that have been executed. Traversing backward through the list and reverse-executing commands cancels their effect; traversing forward and executing commands reexecutes them.
    * An undoable command might have to be copied before it can be placed on the history list. That's because the command object that carried out the original request, say, from a Menultem, will perform other requests at later times. Copying is required to distinguish different invocations of the same command if its state can vary across invocations.
    * For example, a DeleteCommand that deletes selected objects must store different sets of objects each time it's executed. Therefore the DeleteCommand object must be copied following execution, and the copy is placed on the history list. If the command's state never changes on execution, then copying is not required-only a reference to the command need be placed on the history list. Commands that must be copied before being placed on the history list act as prototypes (see Prototype).
3. *Avoiding error accumulation in the undo process*
    * Hysteresis can be a problem in ensuring a reliable, semantics-preserving undo/redo mechanism. Errors can accumulate as commands are executed, unexecuted, and reexecuted repeatedly so that an application's state eventually diverges from original values. It may be necessary therefore to store more information in the command to ensure that objects are restored to their original state. The Memento pattern can be applied to give the command access to this information without exposing the internals of other objects.
4. *Using C++ templates*
    * For commands that (1) aren't undoable and (2) don't require everments, we can use C++ templates to avoid creating a Command subclass for every kind of action and receiver. We show how to do this in the Sample Code section.

### Related Patterns
- A **Composite** can be used to implement MacroCommands.
- A **Memento** can keep state the command requires to undo its effect.
- A command that must be copied before being placed on the history list acts as a **Prototype**.


## Consequences
The Command pattern has the following consequences:
1. Command decouples the object that invokes the operation from the one that knows how to perform it.
2. Commands are first-class objects.
    * They can be manipulated and extended like any other object.
3. You can assemble commands into a composite command. 
    * In general, composite commands are an instance of the Composite pattern.
4. It's easy to add new Commands,
    * because you don't have to change existing classes.


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}