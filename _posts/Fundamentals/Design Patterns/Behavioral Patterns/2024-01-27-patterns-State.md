---
title: "Design Patterns : State"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, State]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-27
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
State

### Also Known As
Objects for States


## Problem

### Intent
Allow an object to **alter its behavior** when its **internal state changes**. The object will appear to change its class.

### Applicability
Use the **State** pattern in either of the following cases:
- An object's behavior depends on its state, and it must **change its behavior at run-time depending on that state**.
- Operations have large, **multipart conditional statements** that depend on the object's state.
    * This state is usually represented by one or more enumerated constants.
    * Often, several operations will contain this same conditional structure.
        + therefore, you want to **avoid this**


## Solution

### Motivation
Suppose that you want to implement the player's movement system where the player runs, jumps, falls differently based on the player's current state, like grounded, under water.
- The **problem** is that you don't want to use conditional statements.
- A **solution** is to put **each branch of the conditional in a separate class**.
    * This lets you treat the object's state as an object in its own right that can vary independently from other objects.

### Participants
- **Context** (`PlayerMovement`)
    * defines the interface of interest to clients.
    * maintains an instance of a ConcreteState subclass that defines the current state.
- **State** (`PlayerState`)
    * defines an interface for encapsulating the behavior associated with a particular state of the Content.
- **ConcreteState subclasses** (`PlayerStateAir`, `PlayerStateGround`, `PlayerStateWater`)
    * each subclass implements a behavior associated with a state of the Context.

### Sample Code
```c++
class PlayerState;
class GM;
// Context
class PlayerMovement {
	friend class PlayerState;
	friend class GM;		// this class exists for test purpose only
public:
	PlayerMovement();

	void run();
	void jump();
	void fall();
private:
	void setState(PlayerState* targetState) {
		state = targetState;
	}
	PlayerState* state;
};


// State
class PlayerState {
public:
	virtual ~PlayerState() = default;

	// these methods have empty definition because derived classes might not use some of them; therefore they should not be pure virtuals
	virtual void run(PlayerMovement* player) { 
		std::cout << "player can't run" << std::endl;	// this exits for test purpose only; this should be empty
	}
	virtual void jump(PlayerMovement* player) { 
		std::cout << "player can't jump" << std::endl;	// this exits for test purpose only; this should be empty
	}
	virtual void fall(PlayerMovement* player) {
		std::cout << "player can't fall" << std::endl;	// this exits for test purpose only; this should be empty
	}
protected:
	void setState(PlayerMovement* player, PlayerState* targetState) {
		player->setState(targetState);
	}
};

void PlayerMovement::run() {
	state->run(this);
}
void PlayerMovement::jump() {
	state->jump(this);
}
void PlayerMovement::fall() {
	state->fall(this);
}


// ConcreteState
class PlayerStateAir : public PlayerState {
public:
	static PlayerStateAir* getInstance();

	void jump(PlayerMovement* player) override;
	void fall(PlayerMovement* player) override;
protected:
	PlayerStateAir() = default;
private:
	static PlayerStateAir* instance;
};
PlayerStateAir* PlayerStateAir::instance = nullptr;
PlayerStateAir* PlayerStateAir::getInstance() {
	if (instance == nullptr)
		instance = new PlayerStateAir();
	return instance;
}
void PlayerStateAir::jump(PlayerMovement* player) {
	/*
		if check whether the player reaches the target y position
			player->fall();
		else
			keep increasing the y position
	*/
	std::cout << "player is jumping through the air" << std::endl;	// this exits for test purpose only
}
void PlayerStateAir::fall(PlayerMovement* player) {
	/*
		if check whether the player is grounded then
			setState(player, PlayerStateGrounded::getInstance());
		else
			keep falling
	*/
	std::cout << "player is falling through the air" << std::endl;	// this exits for test purpose only
}

class PlayerStateGround : public PlayerState {
public:
	static PlayerStateGround* getInstance();

	void run(PlayerMovement* player) override;
	void jump(PlayerMovement* player) override;
protected:
	PlayerStateGround() = default;
private:
	static PlayerStateGround* instance;
};
PlayerStateGround* PlayerStateGround::instance = nullptr;
PlayerStateGround* PlayerStateGround::getInstance() {
	if (instance == nullptr)
		instance = new PlayerStateGround();
	return instance;
}
void PlayerStateGround::run(PlayerMovement* player) {
	/*
		if check whether the player is facing left
			run towards left
		else
			run towards right
	*/
	std::cout << "player is runnnig on the ground" << std::endl;	// this exits for test purpose only
}
void PlayerStateGround::jump(PlayerMovement* player) {
	/*
		setState(player, PlayerStateAir::getInstance());
	*/
	std::cout << "player just jumped on the ground" << std::endl;	// this exits for test purpose only
}

class PlayerStateWater : public PlayerState {
public:
	static PlayerStateWater* getInstance();

	void run(PlayerMovement* player) override;
	void jump(PlayerMovement* player) override;
	void fall(PlayerMovement* player) override;
protected:
	PlayerStateWater() = default;
private:
	static PlayerStateWater* instance;
};
PlayerStateWater* PlayerStateWater::instance = nullptr;
PlayerStateWater* PlayerStateWater::getInstance() {
	if (instance == nullptr)
		instance = new PlayerStateWater();
	return instance;
}
void PlayerStateWater::run(PlayerMovement* player) {
	/*
		if check whether the player is facing left
			move the player left
		else
			move the player right
	*/
	std::cout << "player is moving horizontally under water" << std::endl;	// this exits for test purpose only
}
void PlayerStateWater::jump(PlayerMovement* player) {
	/*
		move the player up
	*/
	std::cout << "player is moving up under water" << std::endl;	// this exits for test purpose only
}
void PlayerStateWater::fall(PlayerMovement* player) {
	/*
		move the player down
	*/
	std::cout << "player is moving down under water" << std::endl;	// this exits for test purpose only
}



PlayerMovement::PlayerMovement() : state(nullptr) {
	state = PlayerStateGround::getInstance();
}




// this code exists for test purpose only
// this is not related to State pattern
class GM {
public:
	void setState(PlayerMovement& player, PlayerState* targetState) {
		player.setState(targetState);
	}
};




// some codes
PlayerMovement player;
GM gm;

player.run();

gm.setState(player, PlayerStateAir::getInstance());
player.run();

gm.setState(player, PlayerStateWater::getInstance());
player.run();

/*
print result
player is runnnig on the ground
player can't run
player is moving horizontally under water
*/
```

### Implementation
The State pattern raises a variety of implementation issues:
1. *Who defines the state transitions?*
    * The State pattern does not specify which participant defines the criteria for state transitions.
        + If the criteria are fixed, then they can be implemented entirely in the Context.
        + It is generally more flexible and appropriate, however, to let the State subclasses themselves specify their successor state and when to make the transition.
            - This requires adding an interface to the Context that lets State objects set the Context's current state explicitly.
    * Decentralizing the transition logic in this way makes it easy to modify or extend the logic by defining new State subclasses.
    * A disadvantage of decentralization is that one State subclass will have knowledge of at least one other, which introduces implementation dependencies between subclasses.
2. *A table-based alternative*
    * We can use tables to map inputs to state transitions.
        + For each state, a table maps every possible input to a succeeding state.
        + In effect, this approach converts conditional code (and virtual functions, in the case of the State pattern) into a table look-up.
    * The main advantage of tables is their regularity:
        + You can change the transition criteria by modifying data instead of changing program code.
    * There are some disadvantages, however:
        + A table look-up is often less efficient than a (virtual) function call.
        + Putting transition logic into a uniform, tabular format makes the transition criteria less explicit and therefore harder to understand.
        + It's usually difficult to add actions to accompany the state transitions.   
        + The table-driven approach captures the states and their transitions, but it must be augmented to perform arbitrary computation on each transition.
    * The key difference between table-driven state machines and the State pattern can be summed up like this:
        + The State pattern models state-specific behavior,
        + whereas the table-driven approach focuses on defining state transitions.
3. *Creating and destroying State objects*
    * A common implementation trade-off worth considering is whether
        + (1) to create State objects only when they are needed and destroy them thereafter versus
        + (2) creating them ahead of time and never destroying them.
    * The first choice is preferable when the states that will be entered aren't known at run-time, and contexts change state infrequently.
        + This approach avoids creating objects that won't be used, which is important if the State objects store a lot of information.
    * The second approach is better when state changes occur rapidly, in which case you want to avoid destroying states, because they may be needed again shortly.
        * Instantiation costs are paid once up-front, and there are no destruction costs at all.
        * This approach might be inconvenient, though, because the Context must keep references to all states that might be entered.

### Related Patterns
- The **Flyweight** pattern explains when and how State objects can be shared.
- State objects are often **Singleton**.


## Consequences
The State pattern has the following consequences:
1. *It localizes state-specific behavior and partitions behavior for different states*
    * The State pattern puts all behavior associated with a particular state into one object.
    * Because all state-specific code lives in a State subclass,
        + new states and transitions can be added easily by defining new subclasses.
2. *It makes state transitions explicit*
    * When an object defines its current state solely in terms of internal data values, its state transitions have no explicit representation;
        + they only show up as assignments to some variables.
    * Introducing separate objects for different states makes the transitions more explicit.
    * Also, State objects can protect the Context from inconsistent internal states, 
    because state transitions are atomic from the Context's perspective
        + they happen by rebinding one variable (the Context's State object variable), not several.
3. *State objects can be shared*
    * If State objects have no instance variables (that is, the state they represent is encoded entirely in their type)
        + then contexts can share a State object.
    * When states are shared in this way, they are essentially flyweights (see **Flyweight**) with no intrinsic state, only behavior.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}