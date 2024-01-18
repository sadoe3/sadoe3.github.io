---
title: "Design Patterns : Abstract Factory"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Creational Pattern, Abstract Factory]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-17
---

# Creational Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Abstract Factory

### Also Known As
Kit


## Problem

### Intent
Provide an **interface** for creating **families** of related or dependent objects **without** specifying their **concrete classes**

### Applicability
Use the **Abstract Factory** pattern when
- a system should be independent of how its products are created, composed, and represented.
- a system should be configured with one of **multiple families (standards)** of products.
- **a family** of related product objects is designed to be used **together**, and you need to enforce this constraint.
- you want to provide a class library of products, and you want to reveal just their interfaces, not their implementations.


## Solution

### Motivation
Suppose that you want to implement the dark mode and light mode (**multiple standards**) for all widgets in your application; plus, you want the application to be switched efficiently.
- In this case, you can solve this problem by defining an abstract `WidgetFactory` class that declares an interface for creating each basic kind of widget.
    * There's also an abstract class for each kind of widget,
    * and concrete subclasses implement widgets for specific modes.
- `WidgetFactory`'s interface has an operation that returns a new widget object for each abstract widget class.
    * Clients call these operations to obtain widget instances,
    * but clients aren't aware of the concrete classes they're using.
    * Thus clients stay independent of the prevailing look and feel.
- There is a concrete subclass of `WidgetFactory` for each mode.
    * Each subclass implements the operations to create the appropriate widget for the mode.
        + For example, the `createScrollBar()` operation on the `DarkWidgetfactory` instantiates and returns a Dark scroll bar, while the corresponding operation on the `LightWidgetFactory` returns a Light scroll bar.
- Clients create widgets solely through the `WidgetFactory` interface and have no knowledge of the classes that implement widgets for a particular mode.
    * In other words, clients only have to commit to an interface defined by an abstract class, not a particular concrete class.
- A `WidgetFactory` also enforces dependencies between the concrete widget classes.
    * A Dark scroll bar should be used with a Dark button and a Dark text editor, and that constraint is enforced automatically as a consequence of using a `DarkWidgetFactory`.

### Participants
- **AbstractFactory** (`WidgetFactory`)
    * declares an interface for operations that create abstract product objects.
- **ConcreteFactory** (`DarkWidgetFactory`, `LightWidgetFactory`)
    * implements the operations to create concrete product objects.
- **AbstractProduct** (`Button`, `ScrollBar`)
    * declares an interface for a type of product object.
- **ConcreteProduct** (`LightButton`, `DarkScrollBar`)
    * defines a product object to be created by the corresponding concrete factory.
    * implements the AbstractProduct interface.
- **Client**
    * uses only interfaces declared by AbstractFactory and AbstractProduct classes.

### Sample Code
```c++
class Button {
	/* some definition */
};
class ScrollBar {
	/* some definition */
};

class WidgetFactory {
public:
	WidgetFactory() = default;

	virtual Button* makeButton() const = 0;
	virtual ScrollBar* makeScrollBar() const = 0;
};



class DarkButton : public Button {
	/* some definition */
};
class DarkScrollBar : public ScrollBar {
	/* some definition */
};

class DarkWidgetFactory : public WidgetFactory {
public:
	DarkWidgetFactory();		// defined somewhere

	Button* makeButton() const override {
		return new DarkButton();
	}
	ScrollBar* makeScrollBar() const override {
		return new DarkScrollBar();
	}
};


class LightButton : public Button {
	/* some definition */
};
class LightScrollBar : public ScrollBar {
	/* some definition */
};

class LightWidgetFactory : public WidgetFactory {
public:
	LightWidgetFactory();		// defined somewhere

	Button* makeButton() const override {
		return new LightButton();
	}
	ScrollBar* makeScrollBar() const override {
		return new LightScrollBar();
	}
};



// some codes
DarkWidgetFactory darkFactory;
application.render(darkFactory);    // render as Dark Theme
```

### Implementation
Here are some useful **techniques** for implementing the Abstract Factory pattern.
1. *Factories as singletons*
    * An application typically needs only one instance of a `ConcreteFactory` per product family.
        + So it's usually best implemented as a **Singleton**.
2. *Creating the products*
    * `AbstractFactory` only declares an interface for creating products. It's up to `ConcreteProduct` subclasses to actually create them.
        + The most common way to do this is to define a factory method (see **Factory Method**) for each product.
        + A concrete factory will specify its products by overriding the factory method for each.
            - While this implementation is simple, it requires a new concrete factory subclass for each product family, even if the product families differ only slightly.
    * If many product families are possible, the concrete factory can be implemented using the **Prototype** pattern.
        + The concrete factory is initialized with a prototypical instance of each product in the family, and it creates a new product by cloning its prototype.
        + The Prototype-based approach eliminates the need for a new concrete factory class for each new product family.
3. *Defining extensible factories*
    * `AbstractFactory` usually defines a different operation for each kind of product it can produce.
        + The kinds of products are encoded in the operation signatures.
        + Adding a new kind of product requires changing the `AbstractFactory` interface and all the classes that depend on it.
    * A more flexible but less safe design is to add a parameter to operations that create objects.
        + This parameter specifies the kind of object to be created.
        + It could be a class identifier, an integer, a string, or anything else that identifies the kind of product.
        + In fact with this approach, `AbstractFactory` only needs a single "Make" operation with a parameter indicating the kind of object to create.
            - The implementation section of **Factory Method** shows how to implement such parameterized operations in C++.
    * But even when no coercion is needed, an inherent problem remains:
        + All products are returned to the client with the same abstract interface as given by the return type.
        + The client will not be able to differentiate or make safe assumptions about the class of a product.
        + If clients need to perform subclass-specific operations, they won't be accessible through the abstract interface.
        + Although the client could perform a downcast (eg., with `std::dynamic_cast` in C++), that's not always feasible or safe,
            - because the downcast can fail. This is the classic trade-off for a highly flexible and extensible interface.

### Related Patterns
- AbstractFactory classes are often implemented with factory methods (**Factory Method**), but they can also be implemented using **Prototype**.
- A concrete factory is often a singleton (**Singleton**).


## Consequences
The Abstract Factory pattern has the following **benefits** and **liabilities**:
1. *It isolates concrete classes*
    * Because a factory encapsulates the responsibility and the process of creating product objects, it isolates clients from implementation classes.
        + Clients manipulate instances through their abstract interfaces which means they don't need to know regarding the details.
2. *It makes exchanging product families easy*
    * The class of a concrete factory appears only once in an application - that is, where it's instantiated.
        + This makes it easy to change the concrete factory an application uses.
        + It can use different product configurations simply by changing the concrete factory.
3. *It promotes consistency among products*
    * When product objects in a family are designed to work together, it's important that an application use objects from only one family at a time.
        + `AbstractFactory` makes this easy to enforce.
4. *Supporting new kinds of products is difficult*
    * Extending abstract factories to produce new kinds of Products isn't easy.
        + That's because the `AbstractFactory` interface fixes the set of products that can be created.
        + Supporting new kinds of products requires extending the factory interface, which involves changing the `AbstractFactory` class and all of its subclasses.
        + We discuss one solution to this problem in the [**Implementation**](https://sadoe3.github.io/design-patterns/patterns-AbstractFactory/#implementation) section.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}