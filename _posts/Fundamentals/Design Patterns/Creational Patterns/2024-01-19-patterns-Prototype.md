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
        + it's worth noting that they are **not subclasses** of `WidgetPrototypeFactory`
        + they are just objects created by `WidgetPrototypeFactory`'s **constructor** with **different parameters**
- **Client** (`Application`)
    * creates a new object by asking a prototype to clone itself.

### Sample Code
```c++
class Button {
public:
	Button() = default;
	Button(const Button& rhs); // defined somewhere
	virtual Button* clone() const = 0;
	// other members
};
class ScrollBar {
public:
	ScrollBar() = default;
	ScrollBar(const ScrollBar& rhs); // defined somewhere
	virtual ScrollBar* clone() const = 0;
	// other members
};

class WidgetFactory {
public:
	WidgetFactory() = default;

	virtual Button* makeButton() const = 0;
	virtual ScrollBar* makeScrollBar() const = 0;
};

class WidgetPrototypeFactory : public WidgetFactory {
public:
	WidgetPrototypeFactory(Button *inputButton, ScrollBar *inputScrollBar) : prototypeButton(inputButton), prototypeScrollBar(inputScrollBar) {}
	~WidgetPrototypeFactory() {
		delete prototypeButton;
		delete prototypeScrollBar;
	}

	Button* makeButton() const override {
		return prototypeButton->clone();
	}
	ScrollBar* makeScrollBar() const override {
		return prototypeScrollBar->clone();
	}
private:
	Button *prototypeButton;
	ScrollBar *prototypeScrollBar;
};


class DarkButton : public Button {
public:
	DarkButton() = default;
	DarkButton(const DarkButton& rhs);			// defined somewhere
	Button* clone() const override {
		return new DarkButton(*this);			// cloning might choose different way from copy construction
	}
	// other members
};
class DarkScrollBar : public ScrollBar {
public:
	DarkScrollBar() = default;
	DarkScrollBar(const DarkScrollBar& rhs);	// defined somewhere
	ScrollBar* clone() const override {
		return new DarkScrollBar();
	}
	/* some definition */
};

class LightButton : public Button {
public:
	LightButton() = default;
	LightButton(const LightButton& rhs);		// defined somewhere
	Button* clone() const override {
		return new LightButton(*this);
	}
	// other members
};
class LightScrollBar : public ScrollBar {
public:
	LightScrollBar() = default;
	LightScrollBar(const LightScrollBar& rhs);	// defined somewhere
	ScrollBar* clone() const override {
		return new LightScrollBar();
	}
	/* some definition */
};


// same definition
class Application {
public:
	void render(const WidgetFactory& factory) {
		Button *newButton = factory.makeButton();
		// render button properly
		ScrollBar* newScrollBar= factory.makeScrollBar();
		// render scroll bar properly
	}
	// other members
};



// some codes
Application application;
WidgetPrototypeFactory mixedWidgetFactory(new DarkButton(), new LightScrollBar());
application.render(mixedWidgetFactory);         // render as mixed Theme
```

### Implementation
Prototype is particularly useful with static languages like C++, where classes are not objects, and little or no type information is available at run-time. 
- Consider the following issues when implementing prototypes:
1. *Using a prototype manager*
    * When the **number** of prototypes in a system **isn't fixed**
        + (that is; they can be created and destroyed dynamically)
        + keep a **registry** of available prototypes.
    * **Clients won't manage** prototypes themselves but will store and retrieve them from the registry.
        + A client will ask the registry for a prototype before cloning it.
        + We call this **registry** a **prototype manager**.
    * A prototype manager is an associative store that returns the prototype matching a given **key**.
        + It has operations for **registering** a prototype under a key and for **unregistering** it at **run-time**.
2. *Implementing the Clone operation*
    * The **hardest** part of the Prototype pattern is implementing the `clone()` operation correctly.
        + It's particularly tricky when object structures contain **circular references**.
    * C++ provides a **copy constructor**.
        + But these facilities don't solve the "*shallow copy versus deep copy*" **problem**
        + That is, does cloning an object in turn **clone** its instance variables, or do the clone and original just **share** the variables?
    * A **shallow copy** is simple and often sufficient.
        + The default copy constructor in C++ does a memberwise copy, which means pointers will be shared between the copy and the original.
    * But cloning prototypes with complex structures usually requires a **deep copy**, because the clone and the original must be independent.
        + Therefore you must ensure that the clone's components are clones of the prototype's components.
        + Cloning forces you to decide what if anything will be shared.
    * If objects in the system provide **Save and Load operations**, then you can use them to provide a default implementation of `clone()` simply by saving the object and loading it back immediately.
        + The **Save** operation saves the object into a memory buffer,
        + and **Load** creates a duplicate by reconstructing the object from the buffer.
3. *Initializing clones*
    * While some clients are perfectly happy with the clone as is, others will want to **initialize** some or all of its internal state to values of their choosing.
    * It might be the case that your prototype classes already define operations for (re)setting key pieces of state.
        + If so, clients may use these operations immediately after cloning.
        + If not, then you may have to introduce an `initialize()` operation that takes initialization parameters as arguments and sets the clone's internal state accordingly.

### Related Patterns
- Prototype and **Abstract Factory** are competing patterns in some ways.
    * They can also be used together, however.
    * An Abstract Factory might store a set of prototypes from which to clone and return product objects.
- Designs that make heavy use of the **Composite** and **Decorater** patterns often can benefit from Prototype as well.


## Consequences
Prototype has many of the same consequences that Abstract Factory and Builder have:
- It hides the concrete product classes from the client
    * thereby reducing the number of names clients know about.
    * Moreover, these patterns let a client work with application-specific classes without modification.
- Additional benefits of the Prototype pattern are listed below.
1. *Adding and removing products at run-time*
    * Because a client can install and remove prototypes at **run-time**, that's a bit more flexible than other creational patterns.
2. *Specifying new objects by varying values*
    * Highly dynamic systems let you define new behavior through object **composition** and **not by defining new classes**.
3. *Specifying new objects by varying structure*
    * Many applications build objects from parts and subparts.
    * Editors for circuit design, for example, build circuits out of subcircuits.
        + For convenience, such applications often let you instantiate complex, user-defined structures, say, to use a specific subcircuit again and again.
    * The Prototype pattern supports this as well.
        + We simply add this subcircuit as a prototype to the palette of available circuit elements.
        + As long as the composite circuit object implements `clone()` as a deep copy, circuits with **different structures can be prototypes**.
4. *Reduced subclassing*
    * Factory Method often produces a hierarchy of `Creator` classes that parallels the product class hierarchy.
    * The Prototype pattern lets you clone a prototype instead of asking a factory method to make a new object.
        + Hence you do **not need a `Creator` class hierarchy** at all.
- The main **liability** of the Prototype pattern is that each subclass of Prototype must implement the `clone()` operation, which may be difficult.
    * For example, adding `clone()` is difficult when the classes under consideration already exist.
    * Implementing `clone()` can be difficult when their internals include objects that don't support copying or have circular references.


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}