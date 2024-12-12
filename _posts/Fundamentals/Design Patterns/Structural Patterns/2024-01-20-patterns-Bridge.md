---
title: "Design Patterns : Bridge"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Structural Pattern, Bridge]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-20
---

# Structural Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Bridge

### Also Known As
Handle/Body


## Problem

### Intent
**Decouple** an abstraction from its implementation so that the **two can vary independently**.

### Applicability
Use the **Bridge** pattern when
- you want to **avoid a permanent binding** between an **abstraction** and its **implementation**.
    * This might be the case, for example, when the **implementation** must be selected or switched at **run-time**.
- both the abstractions and their implementations should be **extensible by subclassing**.
    * In this case, the Bridge pattern lets you combine the different abstractions and implementations and extend them independently.
- **changes** in the **implementation** of an abstraction should have **no impact on clients**
    * That is, their code should **not** have to be **recompiled**.
- (C++) you want to **hide** the **implementation** of an abstraction **completely** from **clients**.
    * In C++ the representation of a class is **visible** in the class **interface**.
- you want to **share** an **implementation** among **multiple objects** (perhaps using reference counting), and this **fact** should be **hidden from the client**.

## Solution

### Motivation
Suppose that you want to implement multiple levels of stage and initialize them differently based on whether the player uses sword or bow.
- We can solve this by using **inheritance** for the `Stage` classes
- However, the **problem** is that the base class of `Stage` class hierarchy needs **two** methods which do the **same** thing because stages for the sword player should be initialized differently from the stages for the bow player
    * plus, the **client** code should **know** the details of the **implementaion**
- A **solution** is to put the interface(abstaction) and its implementation in **separate class hierarchies**
    * we refer to the **relationship** between the interface's base class and implementation's base class as a **bridge**, because it **bridges** the abstraction and its implemenation, letting them **vary independetly** 

### Participants
- **Abstraction** (`Stage`)
    * defines the abstraction's interface.
    * maintains a reference to an object of type Implementor.
- **Refined Abstraction** (`HardStage`, `EasyStage`)
    * Extends the interface defined by Abstraction.
- **Implementor** (`StageImpl`)
    * defines the interface for implementation classes. This interface doesn't have to correspond exactly to Abstraction's interface;
    * in fact the two interfaces can be quite different.
        + That is, the level of abstraction can be different among those two interfaces
        + Typically the Implementor interface provides only primitive operations, and Abstraction defines higher-level operations based on these primitives.
- **Concretelmplementor** (`SwordStageImpl`, `BowStageImpl`)
    * implements the Implementor interface and defines its concrete implementation.


### Sample Code
```c++
class StageImpl;
enum class PlayerType { Sword, Bow };

class StageSystemFactory {
public:
	static StageImpl* getInstance();
	static PlayerType type;
private:
	static StageImpl* instance;
};
StageImpl* StageSystemFactory::instance = nullptr;
PlayerType StageSystemFactory::type = PlayerType::Sword;	// default mode is Sword; this can be altered



// implemenation class hierarchy
// Implementor should be abstract base class
class StageImpl {
public:
	virtual void spawnMonster(const unsigned& monsterCount) = 0;
	// other members
};
// implementation for sword players
class SwordStageImpl : public StageImpl {
public:
	void spawnMonster(const unsigned& monsterCount) override {
		// spawn monsters near the player
		// test message
		std::cout << "monster count : " << monsterCount << "\nmonsters have been spawned near the player" << std::endl;
	}
	// other members
};
// implementation for bow players
class BowStageImpl : public StageImpl {
public:
	void spawnMonster(const unsigned& monsterCount) override {
		// spawn monsters far from the player
		// test message
		std::cout << "monster count : " << monsterCount << "\nmonsters have been spawned far from the player" << std::endl;
	}
	// other members
};


StageImpl* StageSystemFactory::getInstance() {
	if (instance == nullptr) {
		switch (type) {
		case PlayerType::Sword:
			instance = new SwordStageImpl();
			break;
		case PlayerType::Bow:
			instance = new BowStageImpl();
			break;
		}
	}
	return instance;
}



// interface class hierarchy
// default level
class Stage {
public:
	virtual void initializeStage() {
		getImplementor()->spawnMonster(monsterCount);
		// other initialization process
	}
	// other members
protected:
	StageImpl* getImplementor() {
		if (implementor == nullptr)
			implementor = StageSystemFactory::getInstance();
		return implementor;
	}
	unsigned monsterCount = 20;
private:
	StageImpl* implementor;
};
// hard level
class HardStage : public Stage {
public:
	void initializeStage() override {
		getImplementor()->spawnMonster(monsterCount + penaltyMonsterCount);
		// other initialization process
	}
private:
	unsigned penaltyMonsterCount = 30;
	// other data members for hard stage
};
// easy level
class EasyStage : public Stage {
public:
	void initializeStage() override {
		getImplementor()->spawnMonster(monsterCount - advantageMonsterCount);
		// other initialization process
	}
private:
	unsigned advantageMonsterCount = 10;
	// other data members for Easy stage
};



// some codes
HardStage hard;
StageSystemFactory::type = PlayerType::Bow;
hard.initializeStage();

//it would print
//monster count : 50
//monsters have been spawned far from the player
```

### Implementation
Consider the following implementation issues when applying the Bridge pattern:
1. *Only one Implementor*
    * Although there's a **one-to-one relationship** between Abstraction and Implementor, this separation is still **useful** when a **change** in the **implementation** of a class must **not affect** its existing **clients**
        + that is, they shouldn't have to be recompiled, just relinked.
    * In C++, the class interface of the Implementor class can be defined in a **private header file** that isn't provided to clients.
        + This lets you hide an implementation of a class completely from its clients.
2. *Creating the right Implementor object*
    * If Abstraction knows about all Concretelmplementor classes, then it can instantiate one of them in its constructor;
        + it can decide between them based on parameters passed to its constructor.
    * If for example, a collection class supports multiple implementations, the decision can be based on the size of the collection.
        + A linked list implementation can be used for small collections and a hash table for larger ones.
    * Another approach is to choose a default implementation initially and change it later according to usage.
        + For example, if the collection grows bigger than a certain threshold, then it switches its implementation to one that's more appropriate for a large number of items.
    * It's also possible to delegate the decision to another object altogether.
        + In the above example, we can introduce a factory object (see Abstract Factory) whose sole duty is to encapsulate platform-specifics.
        + The factory knows what kind of `StageImpl` object to create for the platform in use; a `Stage` simply asks it for a `StageImpl`, and it returns the right kind.
        + A benefit of this approach is that Abstraction is not coupled directly to any of the Implementor classes.
3. *Sharing implementers*
    * The Body(implementation) stores a **reference count** that the Handle(interface) class increments and decrements.
    * The code for assigning handles with shared bodies has the following general form:
        ```c++
        Handle& Handle::operator= (const Handle& other) {
            other.body->ref();
            body->unref();

            if(body->refCount() == 0)
                delete body;
            body = other.body;
            return *this;
        }
        ```
4. *Using multiple inheritance*
    * You can use **multiple inheritance** in C++ to combine an interface with its implementation.
        + For example, a class can inherit publicly from Abstraction and privately from a Concretelmplementor.
    * But because this approach relies on static inheritance, it **binds** an implementation **permanently** to its interface.
        + Therefore you **cannot** implement a **true** Bridge with multiple inheritance at least not in C++.


### Related Patterns
- An **Abstract Factory** can create and configure a particular Bridge.
- The **Adapter** pattern is geared toward making unrelated classes work together.
    * It is usually applied to systems after they're designed.
    * Bridge, on the other hand, is used up-front in a design to let abstractions and implementations vary independently.


## Consequences
The Bridge pattern has the following consequences:
1. *Decoupling interface and implementation*
    * An implementation is **not bound permanently** to an interface.
    * The **implementation** of an abstraction can be **configured at run-time**.
        + It's even possible for an object to change its implementation at run-time.
    * Decoupling Abstraction and Implementor also **eliminates compile-time dependencies** on the implementation.
        * Changing an implementation class doesn't require recompiling the Abstraction class and its clients.
        * This property is essential when you must ensure binary compatibility between different versions of a class library.
2. *Improved extensibility*
    * You can extend the Abstraction and Implementor **hierarchies independently**.
3. *Hiding implementation details from clients*
    * You can **shield** clients from **implementation details**,
        + like the sharing of implementor objects and the accompanying reference count mechanism (if any).

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}