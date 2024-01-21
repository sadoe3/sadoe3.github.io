---
title: "Design Patterns : Memento"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, Memento]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-26
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Memento

### Also Known As
Token


## Problem

### Intent
Without violating encapsulation, capture and externalize an object's internal state so that the object can be restored to this state later.

### Applicability
Use the **Memento** pattern when
- a snapshot of (some portion of) an object's state must be saved so that it can be restored to that state later, and
- a direct interface to obtaining the state would expose implementation details and break the object's encapsulation.


## Solution

### Motivation
Suppose that ~
- The **problem** is that ~
- A **solution** is to ~

### Participants
- **Memento** (`SolverState`)
    * stores internal state of the Originator object. The memento may store as much or as little of the originator's internal state as necessary at its originator's discretion.
    * protects against access by objects other than the originator. Mementos have effectively two interfaces. Caretaker sees a narrow interface to the Memento-it can only pass the memento to other objects. Originator, in contrast, sees a wide interface, one that lets it access all the data necessary to restore itself to its previous state. Ideally, only the originator that produced the memento would be permitted to access the memento's internal state.
- **Originator** (`ConstraintSolver`)
    * creates a memento containing a snapshot of its current internal state.
    * uses the memento to restore its internal state.
- **Caretaker** (undo mechanism)
    * is responsible for the memento's safekeeping.
    * never operates on or examines the contents of a memento.

### Sample Code
```c++

```

### Implementation
Here are two issues to consider when implementing the Memento pattern:
1. *Language support*
    * Mementos have two interfaces: a wide one for originators and a narrow one for other objects. Ideally the implementation language will support two levels of static protection. C++ lets you do this by making the Originator a friend of Memento and making Memento's wide interface private. Only the narrow interface should be declared public. For example:
        ```c++
        class State;

        class Originator {
        public:
            Memento* createMemento();
            void setMemento(const Memento*);
            // ...
        private:
            State* state;   // internal data structures
            // ...
        };

        class Memento {
        public:
            // narrow public interface
            virtual Memento();
        private:
            // private members accessible only to Originator
            friend class Originator;
            Memento();

            void setState(State*);
            State* getState();
            // ...
            State* state;
            // ...
        };
        ```
2. *Storing incremental changes*
    * When mementos get created and passed back to their originator in a predictable sequence, then Memento can save just the incremental change to the originator's internal state.
For example, undoable commands in a history list can use mementos to ensure that commands are restored to their exact state when they're undone (see Command). The history list defines a specific order in which commands can be undone and redone. That means mementos can store just the incremental change that a command makes rather than the full state of every object they affect. In the Motivation example given earlier, the constraint solver can store only those internal structures that change to keep the line connecting the rectangles, as opposed to storing the absolute positions of these objects.

### Related Patterns
- **Command**: Commands can use mementos to maintain state for undoable operations.
- **Iterator**: Mementos can be used for iteration as described earlier.


## Consequences
The Memento pattern has several consequences:
1. *Preserving encapsulation boundaries*
    * Memento avoids exposing information that only an originator should manage but that must be stored nevertheless outside the originator. The pattern shields other objects from potentially complex Originator internals, thereby preserving encapsulation boundaries.
2. *It simplifies Originator*
    * In other encapsulation-preserving designs, Originator keeps the versions of internal state that clients have requested. That puts all the storage management burden on Originator. Having clients manage the state they ask for simplifies Originator and keeps clients from having to notify originators when they're done.
3. *Using mementos might be expensive*
    * Mementos might incur considerable overhead if Originator must copy large amounts of information to store in the memento or if clients create and return mementos to the originator often enough. Unless encapsulating and restoring Originator state is cheap, the pattern might not be appropriate. See the discussion of incrementality in the Implementation section.
4. *Defining narrow and wide interfaces*
    * It may be difficult in some languages to ensure that only the originator can access the memento's state.
5. *Hidden costs in caring for mementos*
    * A caretaker is responsible for deleting the mementos it cares for. However, the caretaker has no idea how much state is in the memento. Hence an otherwise lightweight caretaker might incur large storage costs when it stores mementos.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}