---
title: "Design Patterns : Strategy"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, Strategy]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-27
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Strategy

### Also Known As
Policy


## Problem

### Intent
Define a **family of algorithms**, encapsulate each one, and make them **interchangeable**. Strategy lets the algorithm vary **independently** from clients that use it.

### Applicability
Use the **Strategy** pattern when
- many related classes differ only in their behavior.
    * Strategies provide a way to configure a class with one of many behaviors.
- you need different variants of an algorithm.
    * For example, you might define algorithms reflecting different space/time trade-offs.
    * Strategies can be used when these variants are implemented as a class hierarchy of algorithms.
- an algorithm uses **data that clients shouldn't know about**.
    * Use the Strategy patter to avoid exposing complex, algorithm-specific data structures.
- a class defines many behaviors, and these appear as multiple conditional statements in its operations.
    * **Instead of many conditionals**, move related conditional branches into their own Strategy class.


## Solution

### Motivation
Suppose that you want to implement a attack system where the Player attacks differently based on the player's job.
- The **problem** is that you don't want to use conditional statements.
- A **solution** is to inheritance for facilitating the **dynamic binding**.

### Participants
- **Strategy** (`PlayerJob`)
    * declares an interface common to all supported algorithms.
        + Context uses this interface to call the algorithm defined by a ConcreteStrategy.
- **ConcreteStrategy** (`PlayerJobFighter`, `PlayerJobGunner`, `PlayerJobSwordMaster`)
    * implements the algorithm using the Strategy interface.
- **Context** (`Player`)
    * is configured with a ConcreteStrategy object.
    * maintains a reference to a Strategy object.
    * may define an interface that lets Strategy access its data.

### Sample Code
```c++
// Strategy
class PlayerJob {
public:
	virtual ~PlayerJob() = default;
	
	virtual void attack() = 0;
protected:
	PlayerJob() = default;
};


// ConcreteStrategy
class PlayerJobFighter : public PlayerJob {
public:
	void attack() override {
		std::cout << "Player uses the player's fists to attack" << std::endl;
	}
};
class PlayerJobGunner : public PlayerJob {
public:
	void attack() override {
		std::cout << "Player uses the player's gun to attack" << std::endl;
	}
};
class PlayerJobSwordMaster : public PlayerJob {
public:
	void attack() override {
		std::cout << "Player uses the player's sword to attack" << std::endl;
	}
};


// Context
class Player {
public:
	Player(PlayerJob* inputJob) : job(inputJob) { }
	~Player() {
		delete job;
	}

	void attack() {
		job->attack();
	}
private:
	PlayerJob* job;
};




// some codes
Player fighter(new PlayerJobFighter());
Player gunner(new PlayerJobGunner());
Player swordMaster(new PlayerJobSwordMaster());

fighter.attack();
gunner.attack();
swordMaster.attack();

/*
print result
Player uses the player's fists to attack
Player uses the player's gun to attack
Player uses the player's sword to attack
*/
```

### Implementation
Consider the following implementation issues:
1. *Defining the Strategy and Context interfaces*
    * The Strategy and Context interfaces must give a ConcreteStrategy efficient access to any data it needs from a context, and vice versa.
    * One approach is to have Context pass data in parameters to Strategy operations
        + in other words, take the data to the strategy.
        + This keeps Strategy and Context decoupled.
        + On the other hand, Context might pass data the Strategy doesn't need.
    * Another technique has a context pass itself as an argument, and the strategy requests data from the context explicitly.
        + Alternatively, the strategy can store a reference to its context, eliminating the need to pass anything at all.
            - Either way, the strategy can request exactly what it needs.
        + But now Context must define a more elaborate interface to its data, which couples Strategy and Context more closely.
    * The needs of the particular algorithm and its data requirements will determine the best technique.
2. *Strategies as template parameters*
    * In C++ templates can be used to configure a class with a strategy.
    * This technique is only applicable
        + if (1) the Strategy can be selected at compile-time,
        + and (2) it does not have to be changed at run-time.
    * In this case, the class to be configured (e.g., Context) is defined as a template class that has a Strategy class as a parameter:
        ```c++
        template <class AStrategy>
        class Context {
        public:
            void operation() { theStrategy.doAlgorithm(); }
            //... 
        private:
            AStrategy theStrategy;
        };
        ```
    * The class is then configured with a Strategy class when it's instantiated:
        ```c++
        class MyStrategy {
        public:
            void doAlgorithm();
        };
        Context<Mystrategy> aContext;
        ```
    *  With templates, there's no need to define an abstract class that defines the interface to the Strategy.
    * Using Strategy as a template parameter also lets you bind a Strategy to its Context statically, which can increase efficiency.
3. *Making Strategy objects optional*
    * The Context class may be simplified if it's meaningful not to have a Strategy object.
    * Context checks to see if it has a Strategy object before accessing it.
        + If there is one, then Context uses it normally.
        + If there isn't a strategy, then Context carries out default behavior.
    * The benefit of this approach is that clients don't have to deal with Strategy objects at all unless they don't like the default behavior.

### Related Patterns
- **Flyweight**: Strategy objects often make good flyweights.


## Consequences
The Strategy pattern has the following benefits and drawbacks:
1. *An alternative to subclassing*
    * Encapsulating the algorithm in separate Strategy classes lets you vary the algorithm independently of its context,
        + making it easier to switch, understand, and extend.
2. *Strategies eliminate conditional statements*
    * Encapsulating the behavior in separate Strategy classes eliminates conditional statements for selecting desired behavior.
    * For example, without strategies, the code for breaking text into lines could look like
        ```c++
        void Composition::repair() {
            switch(breakingStrategy) {
                case SimpleStrategy:
                    composeWithSimpleCompositor();
                    break;
                case TeXStrategy:
                    composeWithTeXCompositor();
                    break;
                // ...
            }
            // merge results with existing composition, if necessary
        }
        ```
    * The Strategy pattern eliminates this case statement by delegating the linebreaking task to a Strategy object:
        ```c++
        void Composition::repair() {
            compositor->compose();
            // merge results with existing composition, if necessary
        } 
        ```
    * Code containing many conditional statements often indicates the need to apply the Strategy pattern.
3. *A choice of implementations*
    * Strategies can provide different implementations of the same behavior.
        + The client can choose among strategies with different time and space trade-offs.
5. *Clients must be aware of different Strategies*
    * The pattern has a potential drawback in that a client must understand how Strategies differ before it can select the appropriate one.
    * Clients might be exposed to implementation issues.
        + Therefore you should use the Strategy pattern only when the variation in behavior is relevant to clients.
6. *Communication overhead between Strategy and Context*
    * It's likely that some ConcreteStrategies won't use all the information passed to them through this interface;
        + simple ConcreteStrategies may use none of it.
    * That means there will be times when the context creates and initializes parameters that never get used.
        + If this is an issue, then you'll need tighter coupling between Strategy and Context.
7. *Increased number of objects*
    * Strategies increase the number of objects in an application.
    * Sometimes you can reduce this overhead by implementing strategies as stateless objects that contexts can share.
        + Any residual state is maintained by the context, which passes it in each request to the Strategy object.
        + Shared strategies should not maintain state across invocations.
    * The **Flyweight** pattern describes this approach in more detail.
    

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}