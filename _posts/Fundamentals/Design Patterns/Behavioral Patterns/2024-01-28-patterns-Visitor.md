---
title: "Design Patterns : Visitor"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, Visitor]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-28
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Visitor


## Problem

### Intent
Represent an operation to be performed on the elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates.

### Applicability
Use the **Visitor** pattern when
- an object structure contains many classes of objects with differing interfaces, and you want to perform operations on these objects that depend on their concrete classes.
- many distinct and unrelated operations need to be performed on objects in an object structure, and you want to avoid "polluting" their classes with these operations.
    * Visitor lets you keep related operations together by defining them in one class. 
    * When the object structure is shared by many applications, use Visitor to put operations in just those applications that need them.
- the classes defining the object structure rarely change, but you often want to define new operations over the structure.
    * Changing the object structure classes requires redefining the interface to all visitors, which is potentially costly.
    * If the object structure classes change often, then it's probably better to define the operations in those classes.


## Solution

### Motivation
Suppose that ~
- The **problem** is that ~
- A **solution** is to ~

### Participants
- **Visitor** (`NodeVisitor`)
    * declares a Visit operation for each class of Concrete Element in the object structure. The operation's name and signature identifies the class that sends the Visit request to the visitor. That lets the visitor determine the concrete class of the element being visited. Then the visitor can access the element directly through its particular interface.
- **Concrete Visitor** (`TypeCheckingVisitor`)
    * implements each operation declared by Visitor. Each operation implements a fragment of the algorithm defined for the corresponding class of object in the structure. Concrete Visitor provides the context for the algorithm and stores its local state. This state often accumulates results during the traversal of the structure.
- **Element** (`Node`)
    * defines an Accept operation that takes a visitor as an argument.
- **ConcreteElement** (`AssighmentNode`, `VariableRefNode`)
    * implements an Accept operation that takes a visitor as an argument.
- **ObjectStructure** (`Program`)
    * can enumerate its elements.
    * may provide a high-level interface to allow the visitor to visit its elements.
    * may either be a composite (see Composite) or a collection such as a list or a set.

### Sample Code
```c++

```

### Implementation
Each object structure will have an associated Visitor class. This abstract visitor class declares a VisitConcreteElement operation for each class of ConcreteElement defining the object structure. EachVisit operation on the Visitor declares its argument to be a particular ConcreteElement, allowing the Visitor to access the interface of the ConcreteElement directly. Concrete Visitor classes override each Visit operation to implement visitor-specific behavior for the corresponding ConcreteElement class.
- The Visitor class would be declared like this in C++:
    ```c++
    class Visitor {
    public:
        virtual void visitElementA(ElementA*);
        virtual void visitElementB(ElementB*);

        // and so on for other concrete elements
    protected:
        Visitor();
    };
    ```
- Each class of ConcreteElement implements an Accept operation that calls the matching Visit... operation on the visitor for that ConcreteElement. Thus the operation that ends up getting called depends on both the class of the element and the class of the visitor.
- The concrete elements are delcared as
    ```c++
    class Element {
    public:
        virtual ~Element();
        virtual void accept(Visistor&) = 0;
    protected:
        Element();
    };

    class ElementA : public Element {
    public:
        ElementA();
        void accepet(Visistor &v) override { v.visitElementA(this); } 
    };  

    class ElementB : public Element {
    public:
        ElementB();
        void accepet(Visistor &v) override { v.visitElementB(this); } 
    };  
    ```
- A `CompositeElement` class might implement `accept` like this:
    ```c++
    class CompositeElement : public Element {
    public:
        void accept(Visitor&) override;
    private:
        List<Element*> *children;
    };

    void CompositeElement::accept(Visitor &v) {
        ListIterator<Element*> i(children);

        for(i.first(); !i.isDone(); i.next()) {
            i.currentItem()->accept(v);
        }
        v.visitCompositeElement(this);
    }
    ```
- Here are two other implementation issues that arise when you apply the Visitor pattern:
1. *Double dispatch*
    * Effectively, the Visitor pattern lets you add operations to classes without changing them. Visitors achieves this by using a technique called **double-dispatch**. It's a well-known techniques. In fact, some programming languages support it directly (CLOS, for example). Languages like C++ and smalltalk support **single-dispatch**.
    * In single-dispatch languages, two criteria determine which operation will fulfill a request: the name of the request and the type of receiver. For example, the operation that a GenerateCode request will call depends on the type of node object you ask. In C++, calling GenerateCode on an instance of VariavleRefNode will call VariableRefNode:: GenerateCode (which generates code for a variables reference). Calling GenerateCode on an AssignmentNode will call AssignmentNode:: GenerateCode (which will generate code for an assignment). The operation that gets executed depends both on the kind of request and the type of the receiver.
    * "Double-dispatch" simply means the operation that gets executed depends on the kind of request and the types of two receivers. Accept is a double-dispatch opera-tion. Its meaning depends on two types: the Visitor's and the Element's. Double-dispatching lets visitors request different operations on each class of element."
    * This is the key to the Visitor pattern: The operation that gets executed depends on both the type of Visitor and the type of Element it visits. Instead of binding operations statically into the Element interface, you can consolidate the operations in a Visitor and use Accept to do the binding at run-time. Extending the Element interface amounts to defining one new Visitor subclass rather than many new Element subclasses.
2. *Who is responsible for traversing the object structure?*
    * A visitor must visit each element of the object structure. The question is, how does it get there? We can put responsibility for traversal in any of three places: in the object structure, in the visitor, or in a separate iterator object (see Iterator).
Often the object structure is responsible for iteration. A collection will simply iterate over its elements, calling the Accept operation on each. A composite will commonly traverse itself by having each Accept operation traverse the element's children and call Accept on each of them recursively.
Another solution is to use an iterator to visit the elements. In C++, you could use either an internal or external iterator, depending on what is available and what is most efficient. In Smalltalk, you usually use an internal iterator using do: and a block. Since internal iterators are implemented by the object structure, using an internal iterator is a lot like making the object structure responsible for iteration. The main difference is that an internal iterator will not cause double-dispatching-it will call an operation on the visitor with an element as an argument as opposed to calling an operation on the element with the visitor as an argument.
But it's easy to use the Visitor pattern with an internal iterator if the operation on the visitor simply calls the operation on the element without recursing.
You could even put the traversal algorithm in the visitor, although you'll end up duplicating the traversal code in each Concrete Visitor for each aggregate Concrete Element.
The main reason to put the traversal strategy in the visitor is to implement a particularly complex traversal, one that depends on the results of the operations on the object structure. We'll give an example of such a case in the Sample Code.


### Related Patterns
- **Composite**: Visitors can be used to apply an operation over an object structure defined by the Composite pattern.
- **Interpreter**: Visitor may be applied to do the interpretation.


## Consequences
Some of the benefits and liabilities of the Visitor pattern are as follows:
1. *Visitor makes adding new operations easy*
    * Visitors make it easy to add operations that depend on the components of complex objects. You can define a new operation over an object structure simply by adding a new visitor. In contrast, if you spread functionality over many classes, then you must change each class to define a new operation.
2. *A visitor gathers related operations and separates unrelated ones*
    * Related behavior isn't spread over the classes defining the object structure; it's localized in a visitor.
Unrelated sets of behavior are partitioned, in their own visitor subclasses. That simplifies both the classes defining the elements and the algorithms defined in the visitors. Any algorithm-specific data structures can be hidden in the visitor.
3. *Adding new Concrete Element classes is hard*
    * The Visitor pattern makes it hard to add new subclasses of Element. Each new ConcreteElement gives rise to a new abstract operation on Visitor and a corresponding implementation in every Concrete Visitor class. Sometimes a default implementation can be provided in Visitor that can be inherited by most of the Concrete Visitors, but this is the exception rather than the rule.
So the key consideration in applying the Visitor pattern is whether you are mostly likely to change the algorithm applied over an object structure or the classes of objects that make up the structure. The Visitor class hierarchy can be difficult to maintain when new ConcreteElement classes are added frequently. In such cases, it's probably easier just to define operations on the classes that make up the structure. If the Element class hierarchy is stable, but you are continually adding operations or changing algorithms, then the Visitor pattern will help you manage the changes.
4. *Visiting across class hierarchies*
    * An iterator (see Iterator) can visit the objects in a structure as it traverses them by calling their operations. But an iterator can't work across object structures with different types of elements. For example, the Iterator interface defined on page 254 can access only objects of type Item:
        ```c++
        template <class Item>
        class Iterator {
            // ...
            Item currentItem() const;
        };
        ```
    * This implies that all elements the iterator can visit have a common parent class Item.
    * Visitor does not have this restriction. It can visit objects that don't have a common parent class. You can add any type of object to a Visitor interface. For example, in
        ```c++
        class Visitor {
        public:
            // ...
            void visitMyType(MyType*);
            void visitYourType(YourType*);
        };
        ```
    * MyType and YourType do not have to be related through inheritance at all
5. *Accumulating state*
    * Visitors can accumulate state as they visit each element in the object structure. Without a visitor, this state would be passed as extra arguments to the operations that perform the traversal, or they might appear as global variables.
6. *Breaking encapsulation*
    * Visitor's approach assumes that the ConcreteElement interface is powerful enough to let visitors do their job. As a result, the pattern often forces you to provide public operations that access an element's internal state, which may compromise its encapsulation.


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}