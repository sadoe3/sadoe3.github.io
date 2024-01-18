---
title: "Design Patterns : Builder"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Creational Pattern, Builder]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-18
---

# Creational Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Builder


## Problem

### Intent
Separate the construction of a complex object from its representation so that the **same construction process** can create **different representations**.

### Applicability
Use the **Builder** pattern when
- the **algorithm** for creating a complex object should be **independent** of the parts that make up the object and how they're assembled.
- the construction process must allow **different representations** for the object that s constructed.


## Solution

### Motivation
Suppose that you want to implement an editor for the game world which should be able to build various monsters.
- the **problem** is that the number of possible creation is open-ended
    * hence it should be easy to add a new monster without modifying the editor
- a **solution** is to configure the `WorldEditor` class with a `MonsterBuilder` object that **builds** (creates) a monster for that world
    * note that, however, `MonsterBuilder` class doesn't create the monster itself;
        + the main purpose is just to define an interface for creating monsters
    * the derived classes from `MonsterBuilder` do the actual work

### Participants
- **Director** (`WorldEditor`)
    * constructs an object using the Builder interface.
- **Builder** (`MonsterBuilder`)
    * specifies an abstract interface for creating parts of a Product object.
- **ConcreteBuilder** (`BossMonsterBuilder`, `WeakMonsterBuilder`)
    * constructs and assembles parts of the product by implementing the Builder interface.
    * defines and keeps track of the representation it creates.
- **Product** (`Monster`)
    * represents the complex object under construction.
        + ConcreteBuilder builds the product's internal representation and defines the process by which it's assembled.
    * includes classes that define the constituent parts, including interfaces for assembling the parts into the final result.
    * has only one physical type although there can be various logical types (e.g., `BossMonster`, `WeakMonster`) 

### Sample Code
```c++
class WorldEditor;

class Monster {
	friend class WorldEditor;
private:
	std::string name;
	unsigned health;
	int power = 1;		// default power value
};

class MonsterBuilder {
public:
	virtual void setName(std::string &monsterName) const { }
	virtual void setHealth(unsigned &monsterHealth) const { }
	virtual void setPower(int &monsterPower) const { }
};

class WorldEditor { 
public:
	Monster* createMonster(const MonsterBuilder &builder) {
		Monster *newMonster = new Monster();
		builder.setName(newMonster->name);
		builder.setHealth(newMonster->health);
		builder.setPower(newMonster->power);

		return newMonster;
	}
	// other members
};

class BossMonsterBuilder : public MonsterBuilder {
public:
	void setName(std::string &monsterName) const override { 
		monsterName = "Good Boss Name";
	}
	void setHealth(unsigned &monsterHealth) const override {
		monsterHealth = 200;
	}
	void setPower(int &monsterPower) const override {
		monsterPower = 30;
	}
};

class WeakMonsterBuilder : public MonsterBuilder {
public:
	void setName(std::string &monsterName) const override {
		monsterName = "Slime";
	}
	void setHealth(unsigned &monsterHealth) const override {
		monsterHealth = 20;
	}
	// use default implementation for power value
};



// some codes
WorldEditor editor;
BossMonsterBuilder bossBuilder;

Monster *boss = editor.createMonster(bossBuilder);
delete boss;
```

### Implementation
- Typically there's an abstract `Builder` class that defines an operation for each component that a director may ask it to create.
    * The operations do nothing by default.
    * A `ConcreteBuilder` class overrides operations for components it's interested in creating.
- Here are other implementation issues to consider:
1. *Assembly and construction interface*
    * Builders construct their products in step-by-step fashion.
        + Therefore the `Builder` class interface must be general enough to allow the construction of products for all kinds of concrete builders.
    * But sometimes you might need access to parts of the product constructed earlier.
        + In that case, the builder would return child nodes to the director, which then would pass them back to the builder to build the parent nodes.
2. *Empty methods as default in Builder*
    * In C++, the build methods are **intentionally not declared pure virtual member functions**.
        + They're defined as empty methods instead, letting clients override only the operations they're interested in.


### Related Patterns
- **Abstract Factory** is similar to Builder in that it too may construct complex objects.
    * The primary difference is that the Builder pattern focuses on constructing a complex object step by step.
        + Abstract Factory's emphasis is on families of product objects (either simple or complex).
        + Builder returns the product as a final step, but as far as the Abstract Factory pattern is concerned, the product gets returned immediately.
- A **Composite** is what the builder often builds.


## Consequences
Here are key **consequences** of the Builder pattern:
1. *It lets you vary a product's internal representation*
    * The Builder object provides the director with an abstract interface for constructing the product.
    * The interface lets the builder **hide** the representation and **internal structure** of the product.
    * It also hides **how** the product gets **assembled**.
        + Because the product is constructed through an abstract interface, all you have to do to **change** the product's **internal representation** is define **a new kind of builder**.
2. *It isolates code for construction and representation*
    * Clients needn't know anything about the classes that define the product's internal structure
        + such classes don't appear in Builder's interface.
    * Each ConcreteBuilder contains all the code to create and assemble a particular kind of product.
        + The code is written once; then different Directors can reuse it to build Product variants from the same set of parts.
3. *It gives you finer control over the construction process*
    * Unlike creational patterns that construct products in one shot, the Builder pattern constructs the product step by step under the director's control.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}