---
title: "Design Patterns : Flyweight"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Structural Pattern, Flyweight]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-22
---

# Structural Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Flyweight


## Problem

### Intent
Use sharing to support large numbers of fine-grained objects efficiently.

### Applicability
The **Flyweight** pattern's effectiveness depends heavily on how and where it's used.
- Apply the Flyweight pattern when all of the following are true:
    * An application uses a large number of objects.
    * Storage costs are high because of the sheer quantity of objects.
    * Most object state can be made extrinsic.
    * Many groups of objects may be replaced by relatively few shared objects once extrinsic state is removed.
    * The application doesn't depend on object identity. Since flyweight objects may be shared, identity tests will return true for conceptually distinct objects.


## Solution

### Motivation
Suppose that ~
- The **problem** is that ~
- A **solution** is to ~

### Participants
- **Flyweight** (`Glyph`)
    * declares an interface through which flyweights can receive and act on extrinsic state.
- **ConcreteFlyweight** (`Character`)
    * implements the Flyweight interface and adds storage for intrinsic state, if any. A ConcreteFlyweight object must be sharable. Any state it stores must be intrinsic; that is, it must be independent of the ConcreteFlyweight object's context.
- **Unshared ConcreteFlyweight** (`Row`, `Column`)
    * not all Flyweight subclasses need to be shared. The Flyweight interface enables sharing; it doesn't enforce it. It's common for UnsharedConcreteFlyweight objects to have ConcreteFlyweight objects as children at some level in the flyweight object structure (as the Row and Column classes have).
- **FlyweightFactory**
    * creates and manages flyweight objects.
    * ensures that flyweights are shared properly. When a client requests a flyweight, the FlyweightFactory object supplies an existing instance or creates one, if none exists.
- **Client**
    * maintains a reference to flyweight(s).
    * computes or stores the extrinsic state of flyweight(s).

### Sample Code
```c++

```

### Implementation
Consider the following issues when implementing the Flyweight pattern:
1. *Removing extrinsic state*
    * The pattern's applicability is determined largely by how easy it is to identify extrinsic state and remove it from shared objects. Removing extrinsic state won't help reduce storage costs if there are as many different kinds of extrinsic state as there are objects before sharing. Ideally, extrinsic state can be computed from a separate object structure, one with far smaller storage requirements.
In our document editor, for example, we can store a map of typographic information in a separate structure rather than store the font and type style with each character object. The map keeps track of runs of characters with the same typographic attributes. When a character draws itself, it receives its typographic attributes as a side-effect of the draw traversal. Because documents normally use just a few different fonts and styles, storing this information externally to each character object is far more efficient than storing it internally.
2. *Managing shared objects*
    * Because objects are shared, clients shouldn't instantiate them directly. FlyweightFactory lets clients locate a particular flyweight.
FlyweightFactory objects often use an associative store to let clients look up flyweights of interest. For example, the flyweight factory in the document editor example can keep a table of flyweights indexed by character codes. The manager returns the proper flyweight given its code, creating the flyweight if it does not already exist.
Sharability also implies some form of reference counting or garbage collection to reclaim a flyweight's storage when it's no longer needed. However, neither is necessary if the number of flyweights is fixed and small (e.g., flyweights for the ASCII character set). In that case, the flyweights are worth keeping around permanently.

### Related Patterns
- The Flyweight pattern is often combined with the **Composite** pattern to implement a logically hierarchical structure in terms of a directed-acyclic graph with shared leaf nodes.
- It's often best to implement **State** and **Strategy** objects as flyweights.


## Consequences
Flyweights may introduce run-time costs associated with transferring, finding, and/or computing extrinsic state, especially if it was formerly stored as intrinsic state. However, such costs are offset by space savings, which increase as more flyweights are shared.
- Storage savings are a function of several factors:
    * the reduction in the total number of instances that comes from sharing
    * the amount of intrinsic state per object
    * whether extrinsic state is computed or stored.
- The more flyweights are shared, the greater the storage savings. The savings increase with the amount of shared state. The greatest savings occur when the objects use substantial quantities of both intrinsic and extrinsic state, and the extrinsic state can be computed rather than stored. Then you save on storage in two ways: Sharing reduces the cost of intrinsic state, and you trade extrinsic state for computation time.
- The Flyweight pattern is often combined with the Composite pattern to represent a hierarchical structure as a graph with shared leaf nodes. A consequence of sharing is that flyweight leaf nodes cannot store a pointer to their parent. Rather, the parent pointer is passed to the flyweight as part of its extrinsic state. This has a major impact on how the objects in the hierarchy communicate with each other.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}