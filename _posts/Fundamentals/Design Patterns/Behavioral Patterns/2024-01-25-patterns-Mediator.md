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
Define an object that encapsulates **how** a set of objects **interact**. Mediator promotes **loose coupling** by keeping objects from referring to each other explicitly, and it lets you vary their **interaction independently**.

### Applicability
Use the **Mediator** pattern when
- a set of objects **communicate** in well-defined but **complex ways**.
    * The resulting interdependencies are unstructured and difficult to understand.
- reusing an object is difficult because it refers to and communicates with many other objects.
- a behavior that's distributed between several classes should be customizable **without a lot of subclassing**.


## Solution

### Motivation
Suppose that you want to implement a simple computer system that prints the information of the attached device when it's interacted.
- The **problem** is that you want to **decouple** the attached devices so that they don't know about each other.
- A **solution** is to implement the **intermediary class** that handles the interaction between devices.

### Participants
- **Mediator** (`SystemDirector`)
    * defines an interface for communicating with Colleague objects.
- **ConcreteMediator** (`ComputerSystemDirector`)
    * implements cooperative behavior by coordinating Colleague objects.
    * knows and maintains its colleagues.
- **Colleague classes** (`Mouse`, `Keyboard`, `Monitor`)
    * each Colleague class knows its Mediator object.
    * each colleague communicates with its mediator whenever it would have other wise communicated with another colleague.

### Sample Code
```c++
// this code exists for test purpose only
// this is not related to Mediator pattern
enum class ButtonType { LeftButton, RightButton, MiddleButton, LastButtonType };
struct Position { int x, y; };

class Device;
// Mediator
class SystemDirector {
public:
	virtual ~SystemDirector() = default;
	virtual void updateDeviceInteracted(Device*) = 0;
protected:
	SystemDirector() = default;
	virtual void createDevices() = 0;
};


// Colleague Classes
class Device {
public:
	Device(SystemDirector* inputDirector) : director(inputDirector) { }
	void updateInteracted() {
		director->updateDeviceInteracted(this);
	}
private:
	SystemDirector* director;
};
class Mouse : public Device {
public:
	Mouse(SystemDirector* inputDirector) : Device(inputDirector), buttonType(ButtonType::LastButtonType), position({0,0}) { }
	Position getMousePosition() {
		// update mouse position info
		return position;
	}
	ButtonType getMouseType() {
		// update mouse type info
		return buttonType;
	}
	void click() {
		updateInteracted();
	}
private:
	ButtonType buttonType;
	Position position;
};
class Keyboard : public Device {
public:
	Keyboard(SystemDirector* inputDirector) : Device(inputDirector), key('k') { }
	char getKey() {
		// update keyboard key info
		return key;
	}
	void type() {
		updateInteracted();
	}
private:
	char key;
};
class Monitor : public Device {
public:
	Monitor(SystemDirector* inputDirector) : Device(inputDirector), screen(std::cout) { }
	void print(const std::string & data) {
		// print given data
		screen << data << std::endl;
	}
private:
	std::ostream& screen;
};


// ConcreteMediator
class ComputerSystemDirector : public SystemDirector {
public:
	ComputerSystemDirector() : mouse(nullptr), keyboard(nullptr), monitor(nullptr) {
		createDevices();
	}
	~ComputerSystemDirector() {
		delete mouse;
		delete keyboard;
		delete monitor;
	}
	void updateDeviceInteracted(Device*) override;


	// ---------------------- test methods
	void makeMouseClick() {
		mouse->click();
	}
	void makeKeyboardType() {
		keyboard->type();
	}
	// ----------------------
protected:
	void createDevices() override {
		mouse = new Mouse(this);
		keyboard = new Keyboard(this);
		monitor = new Monitor(this);
	}
private:
	Mouse* mouse;
	Keyboard* keyboard;
	Monitor* monitor;
};
void ComputerSystemDirector::updateDeviceInteracted(Device* target) {
	std::string result;
	if (target == mouse) {
		result = "mouse position ( " + std::to_string(mouse->getMousePosition().x) + ", " + std::to_string(mouse->getMousePosition().y) + " )  mouse button type : " + std::to_string(static_cast<int>(mouse->getMouseType()));
		monitor->print(result);
	}
	else if (target == keyboard) {
		result = "key : " + std::string(1, keyboard->getKey());
		monitor->print(result);
	}
}




// some codes
ComputerSystemDirector computer;

computer.makeKeyboardType();
computer.makeMouseClick();

/*
print result
key : k
mouse position ( 0, 0 )  mouse button type : 3
*/
```

### Implementation
The following implementation issues are relevant to the Mediator pattern:
1. *Omitting the abstract Mediator class*
    * There's **no need** to define an **abstract** Mediator class when colleagues work with only **one mediator**.
    * The abstract coupling that the Mediator class provides lets colleagues work with different Mediator subclasses, and vice versa.
2. *Colleague-Mediator communication*
    * Colleagues have to communicate with their mediator when an event of interest occurs.
    * One approach is to implement the Mediator as an **Observer** using the Observer pattern.
        + Colleague classes act as Subjects, sending notifications to the mediator whenever they change state.
        + The mediator responds by propagating the effects of the change to other colleagues.
    * Another approach defines a specialized notification interface in Mediator that lets colleagues be more direct in their communication.
        + When communicating with the mediator, a colleague passes itself as an argument, allowing the mediator to identify the sender.
        + The **Sample Code** uses this approach.

### Related Patterns
- **Facade** differs from Mediator in that it abstracts a subsystem of objects to provide a more convenient interface. Its protocol is unidirectional
    * that is, Facade objects make requests of the subsystem classes but not vice versa.
    * In contrast, Mediator enables cooperative behavior that colleague objects don't or can't provide, and the protocol is multidirectional.
* Colleagues can communicate with the mediator using the **Observer** pattern.


## Consequences
The Mediator pattern has the following benefits and drawbacks:
1. *It decouples colleagues*
    * A mediator promotes loose coupling between colleagues.
You can vary and reuse Colleague and Mediator classes independently.
2. *It abstracts how objects cooperate*
    * Making mediation an independent concept and encapsulating it in an object lets you focus on how objects interact apart from their individual behavior.
        + That can help clarify how objects interact in a system.
3. *It centralizes control*
    * The Mediator pattern trades complexity of interaction for complexity in the mediator.
    * Because a mediator encapsulates protocols,
        + it can become more complex than any individual colleague.
    * This can make the mediator itself a monolith that's hard to maintain.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}