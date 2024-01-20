---
title: "Design Patterns : Singleton"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Creational Pattern, Singleton]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-19
---

# Creational Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Singleton


## Problem

### Intent
Ensure a class only has **one instance**, and provide a **global** point of **access** to it.

### Applicability
Use the **Singleton** pattern when
- there must be exactly **one** instance of a class, and it must be accessible to clients from a **well-known access point**.
- when the sole instance should be **extensible** by **subclassing**, and clients should be able to use an extended instance **without modifying their code**.


## Solution

### Motivation
Suppose that you want to implement a class which has only **one instance** and has a **global access** for its instance
- The **problem** is that a global variable makes an object accessible, 
    * but it does **not keep** you from **instantiating multiple objects**
- A **solution** is to make the **class itself responsible** for keeping track of its sole instance by using **static members**

### Participants
- **Singleton** (`WidgetFactory`)
    * defines an `getInstance()` operation that lets clients access its unique instance.
        + `getInstance()` is a `static` member function in C++
    * may be responsible for creating its own unique instance.

### Sample Code
```c++
// this sample code is based on the example of the Abstract Factory pattern
// now it has a new feature : Singleton
enum class Mode { Dark, Light };
Mode currentMode = Mode::Dark;

class Button {
public:
	Button(const std::string &rhs) : button(rhs) {}
	std::string button;
};
class ScrollBar {
public:
	ScrollBar(const std::string& rhs) : scrollBar(rhs) {}
	std::string scrollBar;
};


class WidgetFactory {
public:
	static WidgetFactory* getInstance();			// necessary code for Singleton

	virtual Button* makeButton() const = 0;
	virtual ScrollBar* makeScrollBar() const = 0;

protected:
	WidgetFactory() = default;                      // necessary code for Singleton
private:
	static WidgetFactory* instance;					// necessary code for Singleton
};


class DarkButton : public Button {
public:
	DarkButton() : Button("DarkButton") {}
};
class DarkScrollBar : public ScrollBar {
public:
	DarkScrollBar() : ScrollBar("DarkScrollBar") {}
};

class DarkWidgetFactory : public WidgetFactory {
	friend class WidgetFactory;						// necessary code for Singleton
public:
	Button* makeButton() const override {
		return new DarkButton();
	}
	ScrollBar* makeScrollBar() const override {
		return new DarkScrollBar();
	}

protected:
	DarkWidgetFactory() = default;                  // necessary code for Singleton
};


class LightButton : public Button {
public:
	LightButton() : Button("LightButton") {}
};
class LightScrollBar : public ScrollBar {
public:
	LightScrollBar() : ScrollBar("LightScrollBar") {}
};

class LightWidgetFactory : public WidgetFactory {
	friend class WidgetFactory;						// necessary code for Singleton
public:
	Button* makeButton() const override {
		return new LightButton();
	}
	ScrollBar* makeScrollBar() const override {
		return new LightScrollBar();
	}
protected:
	LightWidgetFactory() = default;                 // necessary code for Singleton
};



// necessary code for Singleton
WidgetFactory* WidgetFactory::instance = nullptr;
WidgetFactory* WidgetFactory::getInstance() {
	if (WidgetFactory::instance == nullptr) {
		switch (currentMode) {
		case Mode::Dark:
			WidgetFactory::instance = new DarkWidgetFactory();
			break;
		case Mode::Light:
			WidgetFactory::instance = new LightWidgetFactory();
			break;
		}
	}
	return WidgetFactory::instance;
}


class Application {
public:
	void print(const WidgetFactory& factory) {
		Button* newButton = factory.makeButton();
		// print button for test purpose
        // drawing can be done if this code is replaced with the code to draw the obejct
		std::cout << newButton->button << std::endl;

		ScrollBar* newScrollBar = factory.makeScrollBar();
        // print scroll bar for test purpose
		std::cout << newScrollBar->scrollBar << std::endl;
	}
	// other members
};



// some codes
Application application;

// necessary code for Singleton
WidgetFactory* darkFactory = DarkWidgetFactory::getInstance();
application.print(*darkFactory);               // print Dark objects

// failed to get light factory;
// because there's already dark factory instance
WidgetFactory* lightFactory = LightWidgetFactory::getInstance();		
application.print(*lightFactory);              // still print Dark objects

delete darkFactory;
```

### Implementation
Here are implementation issues to consider when using the Singleton pattern:
1. *Ensuring a unique instance*
    * A common way to do this is to hide the operation that creates the instance behind a `static` member function that guarantees only **one instance** is created.
        + This operation has access to the `static` **variable** that holds the unique instance, and it ensures the variable is initialized with the unique instance before returning its value.
        + This approach ensures that a singleton is created and initialized before its first use.
    * Notice that the **constructor** is `protected`.
        + A client that tries to instantiate Singleton directly will get an error at compile-time.
        + This ensures that only one instance can ever get created.
        + It's worth noting that the **friendship**s between the base class and the derived classes are needed
            - because the base class should be able to create the derived class by calling their constructors which are defined in the `protected` section.
2. *Subclassing the Singleton class*
    * The main issue is not so much defining the subclass but installing its unique instance so that clients will be able to use it.
        + In essence, the variable that refers to the singleton instance must get initialized with an instance of the subclass.
    * The simplest technique is to determine which singleton you want to use in the Singleton's `getInstance()` operation.
        + The code sample above uses `currentMode` variable to determine

### Related Patterns
**Many patterns** can be implemented using the Singleton pattern.


## Consequences
The Singleton pattern has several benefits:
1. *Controlled access to sole instance*
    * Because the Singleton class encapsulates its sole instance, it can have **strict control** over how and when clients access it.
2. *Reduced name space*
    * The Singleton pattern is an improvement over global variables.
    * It **avoids polluting the name space** with **global variables** that store sole instances.
3. *Permits refinement of operations and representation*
    * The Singleton class may be **subclassed**, and it's easy to configure an application with **an instance** of this extended class.
    * You can configure the application with an instance of the class you need at **run-time**.
    * It's worth noting that the **whole classes** of the same class **hierarchy** do  **share** the `static` members of their base class
        * which means that if **one instance** is **created** from a certain derived class, then it's **not possible** for **other classes** to be created through `getInstace()` method.
4. *Permits a variable number of instances*
    * The pattern makes it easy to change your mind and **allow more than one instance** of the Singleton class.
    * Moreover, you can use the **same approach** to control the number of instances that the application uses.
        + Only the operation that grants access to the Singleton instance needs to change.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}