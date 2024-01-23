---
title: "Design Patterns : Decorator"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Structural Pattern, Decorator]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-21
---

# Structural Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Decorator

### Also Known As
Wrapper


## Problem

### Intent
Attach **additional responsibilities** to an object **dynamically**. Decorators provide a flexible **alternative** to **subclassing** for extending functionality.

### Applicability
Use **Decorator**
- to add responsibilities to individual objects **dynamically** and **transparently**, that is, without affecting other objects.
- for **responsibilities** that can be **withdrawn**.
- when extension by subclassing is impractical.
    * Sometimes a large number of independent extensions are possible and would produce an explosion of subclasses to support every combination.
    * Or a class definition may be hidden or otherwise unavailable for subclassing.


## Solution

### Motivation
Suppose that you want to implement the attacking system for the player. Your objective is to attach **additional abilities** to the player object **dynamically**
- The **problem** is that if you utilize basic inheritance, the abilities are attached **statically**.
- A **solution** is to **enclose** the player object in another object that adds the additional ability
    * The enclosing object is called a **decorator**.
    * The decorator **conforms** to the **interface** of the component it decorates so that its presence is **transparent** to the component's clients.
        + Transparency lets you **nest** decorators **recursively**, thereby allowing an **unlimited number** of added responsibilities
    * The decorator forwards requests to the component and may perform additional actions **before or after forwarding**

### Participants
- **Component** (`ObjectComponent`)
    * defines the interface for objects that can have responsibilities added to them dynamically.
- **ConereteComponent** (`Player`)
    * defines an object to which additional responsibilities can be attached.
- **Decorator** (`Decorator`)
    * maintains a reference to a Component object and defines an interface that conforms to Component's interface.
- **ConcreteDecorator** (`DecoratorStun`, `DecoratorBleed`)
    * adds responsibilities to the component.

### Sample Code
```c++
// this code exists for test purpose only
// this is not related to Decorator pattern
class ObjectComponent;
class Monster {
	// monster class
};
class GameWorld {
public:
	void setPlayer(ObjectComponent* targetPlayer) {
		player = targetPlayer;
	}
	void makePlayerAttack(Monster* targetMonster);
	// other members
private:
	ObjectComponent* player;
};




// Component
class ObjectComponent {
public:
	virtual ~ObjectComponent() { std::cout << "ObjectComponent is deleted" << std::endl; }
	virtual void attack(Monster* = nullptr) = 0;
protected:
	Monster* getAttakableMonster() {
		Monster* targetMonster = nullptr;
		// search for the player's attackable area
		return targetMonster;
	}
};

void GameWorld::makePlayerAttack(Monster* targetMonster) {
	if (player != nullptr)
		player->attack(targetMonster);

	std::cout << "\n\n\n";
}



// Concrete Component
class Player : public ObjectComponent {
public:
	~Player() {
		std::cout << "Player is deleted" << std::endl;
	}
	void attack(Monster* targetMonster = nullptr) override {
		if (targetMonster == nullptr)
			targetMonster = getAttakableMonster();
		if (targetMonster == nullptr)
			return;

		damageMonster(targetMonster);
	}
private:
	void damageMonster(Monster* targetMonster) {
		// decrease the health of the monster
		std::cout << "Monster is damaged" << std::endl;
	}
};


// Decorator
class Decorator : public ObjectComponent {
public:
	Decorator(ObjectComponent*targetComponent) : component(targetComponent) { }
	virtual ~Decorator() {
		delete component;
		std::cout << "Decorator is deleted" << std::endl;
	}

	virtual void attack(Monster* targetMonster = nullptr) {
		component->attack(targetMonster);
	}
protected:
	ObjectComponent* component;
};

// Concrete Decorator
class DecoratorStun : public Decorator {
public:
	DecoratorStun(ObjectComponent* targetComponent) : Decorator(targetComponent) { }
	~DecoratorStun() {
		std::cout << "DecoratorStun is deleted" << std::endl;
	}

	void attack(Monster* targetMonster = nullptr) override {
		if (targetMonster == nullptr)
			targetMonster = getAttakableMonster();
		if (targetMonster == nullptr)
			return;

		stunMonster(targetMonster);
		component->attack(targetMonster);
	}
private:
	void stunMonster(Monster* targetMonster) {
		// stun target
		std::cout << "Monster is stunned" << std::endl;
	}
};
class DecoratorBleed : public Decorator {
public:
	DecoratorBleed(ObjectComponent* targetComponent) : Decorator(targetComponent) { }
	~DecoratorBleed() {
		std::cout << "DecoratorBleed is deleted" << std::endl;
	}

	void attack(Monster* targetMonster = nullptr) override {
		if (targetMonster == nullptr)
			targetMonster = getAttakableMonster();
		if (targetMonster == nullptr)
			return;

		component->attack(targetMonster);
		bleedMonster(targetMonster);
	}
private:
	void bleedMonster(Monster* targetMonster) {
		// make target bleed
		std::cout << "Monster is bleeding" << std::endl;
	}
};





// some codes
GameWorld game;
ObjectComponent* player = new DecoratorStun(new DecoratorBleed(new Player));
game.setPlayer(player);
Monster monster;
game.makePlayerAttack(&monster);

delete player;
/*
print result
Monster is stunned
Monster is damaged
Monster is bleeding



DecoratorStun is deleted
DecoratorBleed is deleted
Player is deleted
ObjectComponent is deleted
Decorator is deleted
ObjectComponent is deleted
Decorator is deleted
ObjectComponent is deleted
*/
```

### Implementation
Several issues should be considered when applying the Decorator pattern:
1. *Interface conformance*
    * A decorator object's interface must **conform** to the **interface** of the component it decorates.
        + ConcreteDecorator classes must therefore inherit from a common class (at least in C++).
2. *Omitting the abstract Decorator class*
    * There's **no need** to define an **abstract Decorator class** when you **only** need to add **one responsibility**.
        + In that case, you can **merge** Decorator's responsibility for forwarding requests to the component into the Concrete Decorator.
3. *Keeping Component classes lightweight*
    * To ensure a conforming interface, components and decorators must descend from a common Component class.
        + It's important to keep this common class **lightweight**; that is, it should focus on defining an **interface**, **not** on storing **data**.
    * The definition of the data representation should be deferred to subclasses;
        + otherwise the complexity of the Component class might make the **decorators** too **heavyweight** to use in quantity.
        + Putting **a lot of functionality** into **Component** also increases the probability that concrete **subclasses** will pay for **features they don't need**.
4. *Changing the skin of an object versus changing its guts*
    * We can think of a **decorator** as a **skin** over an object that changes its behavior.
    * An **alternative** is to **change** the object's **guts** (internals).
    * The **Strategy** pattern is a good example of a pattern for changing the guts.
        + Since the Decorator pattern only changes a component from the outside, the component doesn't have to know anything about its decorators; that is, the decorators are transparent to the component
        + With strategies, the component itself knows about possible extensions. So it has to reference and maintain the corresponding strategies
    * Strategies are a **better** choice in situations where the **Component** class is **intrinsically heavyweight**, thereby making the Decorator pattern too **costly** to apply.
        + In the Strategy pattern, the component forwards some of its behavior to a separate strategy object.
        + The Strategy pattern lets us alter or extend the component's functionality by replacing the strategy object.

### Related Patterns
- **Adapter**: A decorator is different from an adapter in that a decorator only changes an object's responsibilities, not its interface;
    * an adapter will give an object a completely new interface.
- **Composite**: A decorator can be viewed as a degenerate composite with only one com-ponent.
    * However, a decorator adds additional responsibilities-it isn't intended for object aggregation.
- **Strategy**: A decorator lets you change the skin of an object; a strategy lets you change the guts.
    * These are two alternative ways of changing an object.


## Consequences
The Decorator pattern has at least two key benefits and two liabilities:
1. *More flexibility than static inheritance*
    * The Decorator pattern provides a more flexible way to add responsibilities to objects than can be had with static (multiple) inheritance.
        + With decorators, responsibilities can be added and removed at run-time simply by attaching and detaching them.
    * Decorators also make it easy to add a property twice. 
2. *A decorator and its component aren't identical*
    * A decorator acts as a transparent enclosure.
    * But from an object identity point of view, a decorated component is not identical to the component itself.
        + Hence you shouldn't rely on object identity when you use decorators.
3. *Lots of little objects*
    * A design that uses Decorator often results in systems composed of lots of little objects that all look alike
        + The objects differ only in the way they are interconnected, not in their class or in the value of their variables.
    + Although these systems are easy to customize by those who understand them, they can be hard to learn and debug.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}