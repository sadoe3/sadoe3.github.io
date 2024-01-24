---
title: "Design Patterns : Flyweight"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Structural Pattern, Flyweight]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-22
---

# Structural Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Flyweight


## Problem

### Intent
Use **sharing** to support **large numbers** of fine-grained **objects** efficiently.

### Applicability
The **Flyweight** pattern's effectiveness depends heavily on how and where it's used.
- Apply the Flyweight pattern when all of the following are true:
    * An application uses a **large number of objects**.
    * **Storage costs are high** because of the enormous quantity of objects.
    * Most object **state** can be made **extrinsic**.
    * Many groups of objects may be replaced by relatively few **shared** objects once extrinsic state is **removed**.
    * The application doesn't depend on object identity.
        + Since flyweight objects may be shared, identity tests will return true for conceptually distinct objects.


## Solution

### Motivation
Suppose that you want to implement a war simulator which needs the enormous quantity of soldiers. 
- The **problem** is that if **each** of soldiers of the **same type** has their **visual** data and **audio** data, the simulator would require **huge memory** which seems impractical.
- A **solution** is to **share** such data which are always same if they are the same type of solider.
    * such sharable data are called **intrinsic state**s
        + for instance, visual data, audio data can be shared
    * the opposite data are called **extrinsic state**s
        + for instance, health, position cannot be shared

### Participants
- **Flyweight** (`SoldierInterface`)
    * declares an interface through which flyweights can receive and act on extrinsic state.
- **ConcreteFlyweight** (`SoldierAppearance`, `SoldierSound`)
    * implements the Flyweight interface and adds storage for intrinsic state, if any. 
    * A ConcreteFlyweight object must be sharable. 
    * Any state it stores must be intrinsic;
        + that is, it must be independent of the ConcreteFlyweight object's context.
- **UnsharedConcreteFlyweight** (`Soldier`)
    * not all Flyweight subclasses need to be shared. 
    * The Flyweight interface enables sharing; 
        + it doesn't enforce it.
    * It's common for UnsharedConcreteFlyweight objects to have ConcreteFlyweight objects as children at some level in the flyweight object structure
- **FlyweightFactory** (`SoldierFactory`)
    * creates and manages flyweight objects.
    * ensures that flyweights are shared properly.
    * When a client requests a flyweight, the FlyweightFactory object supplies an existing instance or creates one, if none exists.
- **Client**
    * maintains a reference to flyweight(s).
    * computes or stores the extrinsic state of flyweight(s).

### Sample Code
```c++
// this code exists for test purpose only
// this is not related to Flyweight pattern
struct Position { int x; int y; };
enum class SoldierType { Infantry, Sniper, Medic, LastSoldierType };
struct Memory { static long long memoryForClean; static long long memoryForBad; };
long long Memory::memoryForClean = 0;
long long Memory::memoryForBad = 0;


// Flyweight
class SoldierInterface {
public:
	virtual ~SoldierInterface() = default;
	virtual void move(const Position& targetPosition) { }
	virtual void attack(SoldierInterface* targetSoldier) { }
	// other members
protected:
	SoldierInterface() = default;
};


// ConcreteFlyweight
class SoldierAppearance : public SoldierInterface {
public:
	SoldierAppearance(const SoldierType& soldierType) {
		// initialize visual data based on the given soldier type
	}
	// other members
private:
	// data members regarding visual data
	double dummy;
	double dummya;
	double dummyb;
	double dummyc;
	double dummyd;
};
class SoldierSound : public SoldierInterface {
public:
	SoldierSound(const SoldierType& soldierType) {
		// initialize audio data based on the given soldier type
	}
	// other members
private:
	// data members regarding audio data
	double dummy;
	double dummya;
	double dummyb;
	double dummyc;
	double dummyd;
};


// UnsharedConcreteFlyweight
class Soldier : public SoldierInterface {
public:
	Soldier(SoldierType& soldierType, SoldierAppearance& visualData, SoldierSound& audioData, const Position& position) : type(soldierType), appearance(visualData), sound(audioData), position(position) { }

	void move(const Position& targetPosition) override { 
		position = targetPosition;
	}
	void attack(SoldierInterface* targetSoldier) override { 
		// attacks based on its type
	}
	// other members
private:
	// intrinsic states
	SoldierType& type;
	SoldierAppearance& appearance;
	SoldierSound& sound;

	// extrinsic states
	Position position;

	// other members
};


// FlyweightFactory
class SoldierFactory {
public:
	SoldierFactory();
	~SoldierFactory();
	SoldierType* getSoldierTypeData(const SoldierType& soldierType) {
		if (typeData[soldierType] == nullptr) {
			typeData[soldierType] = new SoldierType(soldierType);
			Memory::memoryForClean += sizeof(*typeData[soldierType]);
		}
		return typeData[soldierType];
	}
	SoldierAppearance* getSoldierVisualData(const SoldierType& soldierType) {
		if (visualData[soldierType] == nullptr) {
			visualData[soldierType] = new SoldierAppearance(soldierType);
			Memory::memoryForClean += sizeof(*visualData[soldierType]);
		}
		return visualData[soldierType];
	}
	SoldierSound* getSoldierAudioData(const SoldierType& soldierType) {
		if (audioData[soldierType] == nullptr) {
			audioData[soldierType] = new SoldierSound(soldierType);
			Memory::memoryForClean += sizeof(*audioData[soldierType]);
		}
		return audioData[soldierType];
	}
	Soldier* createSoldier(const SoldierType& soldierType, const Position& position) {
		auto newSoldier = new Soldier(*getSoldierTypeData(soldierType), *getSoldierVisualData(soldierType), *getSoldierAudioData(soldierType), position);
		Memory::memoryForClean += sizeof(*newSoldier);
		return newSoldier;
	}
private:
	std::map<SoldierType, SoldierType*> typeData;
	std::map<SoldierType, SoldierAppearance*> visualData;
	std::map<SoldierType, SoldierSound*> audioData;
};
SoldierFactory::SoldierFactory() {
	for (int curType = 0, lastType = static_cast<int>(SoldierType::LastSoldierType); curType != lastType; curType++) {
		typeData[static_cast<SoldierType>(curType)] = nullptr;
		visualData[static_cast<SoldierType>(curType)] = nullptr;
		audioData[static_cast<SoldierType>(curType)] = nullptr;
	}
}
SoldierFactory::~SoldierFactory() {
	for (int curType = 0, lastType = static_cast<int>(SoldierType::LastSoldierType); curType != lastType; curType++) {
		delete typeData[static_cast<SoldierType>(curType)];
		delete visualData[static_cast<SoldierType>(curType)];
		delete audioData[static_cast<SoldierType>(curType)];
	}
}





// Bad Design for Soldier Class
class BadSoldier : public SoldierInterface {
public:
	BadSoldier(const SoldierType& soldierType, const Position& position) : type(soldierType), appearance(soldierType), sound(soldierType), position(position) { 
		Memory::memoryForBad += sizeof(*this);
	}

	void move(const Position& targetPosition) override {
		position = targetPosition;
	}
	void attack(SoldierInterface* targetSoldier) override {
		// attacks based on its type
	}
	// other members
private:
	// intrinsic states; but not shared
	SoldierType type;
	SoldierAppearance appearance;
	SoldierSound sound;

	// extrinsic states
	Position position;

	// other members
};




// some codes
SoldierFactory soldierFactory;
const int TOTAL_COUNT = 1000;

// clean code
std::vector<Soldier*> squadInfantry;
for (int curCount = 0; curCount < TOTAL_COUNT; curCount++)
    squadInfantry.push_back(soldierFactory.createSoldier(SoldierType::Infantry, { 1 + curCount, 1 - curCount }));

std::vector<Soldier*> squadSinper;
for (int curCount = 0; curCount < TOTAL_COUNT; curCount++)
    squadSinper.push_back(soldierFactory.createSoldier(SoldierType::Sniper, { 100 + curCount, 100 - curCount }));

std::cout << "total memory used for clean design : " << Memory::memoryForClean / 1024.0 << "KB" << std::endl;

for (int curCount = 0; curCount < TOTAL_COUNT; curCount++) {
    delete squadInfantry[curCount];
    delete squadSinper[curCount];
}


// bad code
std::vector<BadSoldier*> badSquadInfantry;
for (int curCount = 0; curCount < TOTAL_COUNT; curCount++)
    badSquadInfantry.push_back(new BadSoldier(SoldierType::Infantry, { 1 + curCount, 1 - curCount }));

std::vector<BadSoldier*> badSquadSniper;
for (int curCount = 0; curCount < TOTAL_COUNT; curCount++)
    badSquadSniper.push_back(new BadSoldier(SoldierType::Sniper, { 100 + curCount, 100 - curCount }));

std::cout << "total memory used for bad design : " << Memory::memoryForBad / 1024.0 << "KB" << std::endl;

for (int curCount = 0; curCount < TOTAL_COUNT; curCount++) {
    delete squadInfantry[curCount];
    delete squadSinper[curCount];
}


/*
print result
total memory used for clean design : 47.0703KB
total memory used for bad design : 218.75KB
*/
```

### Implementation
Consider the following issues when implementing the Flyweight pattern:
1. *Removing extrinsic state*
    * The pattern's applicability is **determined** largely by **how easy** it is to **identify extrinsic** state and remove it from **shared objects**.
    * Removing extrinsic state won't help reduce storage costs
        + if there are as many different kinds of extrinsic state as there are objects before sharing.
    * Ideally, extrinsic state can be computed from a separate object structure, one with far smaller storage requirements.
2. *Managing shared objects*
    * Because objects are shared, clients shouldn't instantiate them directly. 
    * FlyweightFactory lets clients locate a particular flyweight.
        + FlyweightFactory objects often use an **associative container** (like `std::map`) to let clients look up flyweights of interest. 
    * Sharability also implies some form of **reference counting** or garbage collection to reclaim a flyweight's storage when it's **no longer needed**.
        + However, they would **not** be **necessary** if the number of flyweights is **fixed and small**.
            - In that case, the flyweights are worth keeping around **permanently**.

### Related Patterns
- The Flyweight pattern is often combined with the **Composite** pattern to implement a logically hierarchical structure in terms of a directed-acyclic graph with shared leaf nodes.
- It's often best to implement **State** and **Strategy** objects as flyweights.


## Consequences
Flyweights may introduce **run-time costs** associated with transferring, finding, and/or computing extrinsic state, especially if it was formerly stored as intrinsic state. 
- However, such costs are **offset by space savings**, which increase as more flyweights are shared.
- Storage savings are a function of several factors:
    * the reduction in the total number of instances that comes from sharing
    * the amount of intrinsic state per object
    * whether extrinsic state is computed or stored.
- The **more** flyweights are **shared**, the **greater** the storage **savings**.
    * The greatest savings occur when the objects use substantial quantities of both intrinsic and extrinsic state,
        + and the **extrinsic** state can be **computed** rather than stored.
    + Then you save on storage in two ways:
        + Sharing reduces the cost of intrinsic state,
        + and you trade extrinsic state for computation time.
- The Flyweight pattern is often combined with the Composite pattern to represent a hierarchical structure as a graph with shared leaf nodes.
    * A consequence of sharing is that flyweight leaf nodes cannot store a pointer to their parent.
        + Rather, the parent pointer is passed to the flyweight as part of its extrinsic state.
    * This has a major impact on how the objects in the hierarchy communicate with each other.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}