---
title: "Design Patterns : Adapter"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Structural Pattern, Adapter]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-20
---

# Structural Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Adapter

### Also Known As
Wrapper


## Problem

### Intent
**Convert the interface** of a class into another interface clients expect. Adapter lets classes **work together** that couldn't **otherwise** because of **incompatible** interfaces.

### Applicability
Use the **Adapter** pattern when
- you want to use an **existing class**, and its **interface does not match** the one you need.
- you want to create a reusable class that **cooperates** with unrelated or unforeseen classes
    * that is, **classes** that **don't necessarily have compatible interfaces**.
- (*object adapter only*) you need to use several existing subclasses, but it's **impractical** to adapt their interface by **subclassing every one**.
    * An object adapter can **adapt** the interface of its **parent class**.


## Solution

### Motivation
Suppose that you implement a shooting game and all `Weapon`s in your game have a same operation `shoot()` which spawns a projectile.
- The **problem** is that you added a new kind of weapon which is a `Sword` class from the **different** library.
    * The `Sword` class doesn't have an operation `shoot()`, however, you want it to have that operation.
- A **solution** is to **define a new class** which **adapts** the **interface** of the `Weapon` class and the **implementation** of `Sword` class
- There are 2 possible solution to implement the **adapter class**
    * **Class Adapter**
        + This approach uses **multiple inheritance** feature to inherit the **interface publicly** and the **implementation privately**
    * **Object Adapter**
        + This approach uses **composition** of the implementation class's instance and **inherits** the interface only

### Participants
- **Target** (`Weapon`)
    * defines the domain-specific interface that Client uses.
- **Adaptee** (`Sword`)
    * defines an existing interface that needs adapting.
- **Adapter** (`WeaponSword`)
    * adapts the interface of Adaptee to the Target interface.
- **Client** (`Player`)
    * collaborates with objects conforming to the Target interface.


### Sample Code
```c++
struct Projectile {
	/*this class handles the projectile*/
};
class Weapon {
public:
	virtual Projectile* shoot() = 0;
	// other members
};


struct ProjectileRifle : public Projectile {
	/*this class handles the projectile from the rifle*/
};
class WeaponRifle : public Weapon {
public:
	Projectile* shoot() override {
		return new ProjectileRifle();
	}
	// other members
};



// new weapon from the different library
class Sword {
public:
	// attack() method lets the player attack towards enemy;
	// but this doesn't spwan any projectile
	void attack() {
		// some definition
	}
	// other members
};


// adapter classes
struct ProjectileSword : public Projectile {
	/*this class handles the projectile from the sword*/
};

// 1. class adapter
class WeaponSword : public Weapon, private Sword {
public:
	Projectile* shoot() override {
		// when the player shoots the sword;
		// the player attacks first and spawn the projectile towards the enemy
		attack();
		return new ProjectileSword();
	}
	// other members
};
// 2. object adapter
class WeaponSword : public Weapon {
public:
	WeaponSword(Sword *inputSword) : sword(inputSword) {}
	~WeaponSword() { delete sword; }
	Projectile* shoot() override {
		// when the player shoots the sword;
		// the player attacks first and spawn the projectile towards the enemy
		sword->attack();
		return new ProjectileSword();
	}
	// other members
private:
	Sword *sword;
};


class Player {
public:
	void equipWeapon(Weapon* inputWeapon) {
		weapon = inputWeapon;
	}
	void shoot() {
		weapon->shoot();
	}
private:
	Weapon* weapon;




// some codes
Player player;

// 1. class adapter approach
WeaponSword *sword = new WeaponSword();
player.equipWeapon(sword);
player.shoot();
delete sword;

// 2. object adapter approach
WeaponSword* sword = new WeaponSword(new Sword());
player.equipWeapon(sword);
player.shoot();
delete sword;
```

### Implementation
Although the implementation of Adapter is usually straightforward, here are some issues to keep in mind:
- **Pluggable adapters**
    * The first step is to find a "**narrow**" interface for Adaptee
        + that is, the **smallest subset** of operations that lets us do the adaptation.
        + A narrow interface consisting of only a couple of operations is easier to adapt than an interface with dozens of operations.
    * The narrow interface leads to 3 implementation approaches:
        + (a) *Using abstract operations*
            - Subclasses must implement the abstract operations and adapt the hierarchically structured object.
        + (b) *Using delegate objects*
            - Statically typed languages like C++ require an explicit interface definition for the delegate. 
        + (c) *Parameterized adapters*
            - The usual way to support pluggable adapters in Smalltalk is to parameterize an adapter with one or more blocks. 
            - The block construct supports adaptation without subclassing.
            - A block can adapt a request, and the adapter can store a block for each individual request.

### Related Patterns
- **Bridge** has a structure similar to an object adapter, but Bridge has a different intent:
    * It is meant to separate an interface from its implementation so that they can be varied easily and independently.
    * An adapter is meant to change the interface of an existing object.
- **Decorator** enhances another object without changing its interface.
    * A decorator is thus more transparent to the application than an adapter is.
    * As a consequence, Decorator supports recursive composition, which isn't possible with pure adapters.
- **Proxy** defines a representative or surrogate for another object and does not change its interface.


## Consequences
Class and object adapters have different trade-offs.
- A **class adapter**
    * adapts Adaptee to Target by committing to a concrete Adapter class.
        + As a consequence, a class adapter **won't work** when we want to **adapt** a class and **all its subclasses**.
    * lets Adapter **override** some of **Adaptee's behavior**, since Adapter is a subclass of Adaptee.
- An **object adapter**
    * lets a single Adapter work with **many Adaptees**
        + that is, the Adaptee itself and **all of its subclasses** (if any).
        + The Adapter can also add **functionality** to **all Adaptees at once**.
    * makes it **harder** to **override Adaptee behavior**.
        + It will require subclassing Adaptee and making Adapter refer to the subclass rather than the Adaptee itself.
- Here are other issues to consider when using the Adapter pattern:
1. *How much adapting does Adapter do?*
    * The amount of work Adapter does **depends** on **how similar** the Target **interface** is to Adaptee's.
2. *Using two-way adapters to provide transparency*
    * A **potential problem** with adapters is that they aren't transparent to all clients.
        + An **singly adapted object** no longer conforms to the Adaptee interface, so it **can't be used** as is wherever an **Adaptee object can**.
    * **Two-way adapters** can provide such transparency.
        + Specifically, they're useful when two different clients need to view an object differently.


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}