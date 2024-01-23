---
title: "Design Patterns : Facade"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Structural Pattern, Facade]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-22
---

# Structural Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Facade


## Problem

### Intent
Provide a **unified interface** to a set of interfaces in a **subsystem**. Facade defines a **higher-level interface** that makes the subsystem easier to use.

### Applicability
Use the **Facade** pattern when
- you want to provide a **simple interface** to a complex subsystem.
    * Subsystems often get more complex as they evolve.
    * Most patterns, when applied, result in more and smaller classes.
        + This makes the subsystem more reusable and easier to customize, but it also becomes harder to use for clients that don't need to customize it.
    * A facade can provide a **simple default view** of the subsystem that is good enough for most clients.
    * Only clients needing more customizability will need to look beyond the facade.
- there are many **dependencies** between clients and the implementation classes of an abstraction.
    * Introduce a facade to **decouple** the subsystem from clients and other subsystems, thereby promoting subsystem independence and **portability**.
- you want to layer your subsystems.
    * Use a facade to define an entry point to each subsystem level.
    * If subsystems are dependent, then you can simplify the dependencies between them by making them communicate with each other solely through their facades.


## Solution

### Motivation
Suppose that you want to implement a game manager class which has a method that deals with the initialization of the stage.
- The **problem** is that if you implement a "God" class which has **all responsibilities** regarding the creation of the stage, this would **not** be a [**clean design**](https://sadoe3.github.io/coding-practice/practice-chapter10/#the-single-responsibility-principle).
- A **solution** is to implement **multiple** classes which have the **single responsibility** respectively, and to implement a class which has a **high-level responsibility** that is to use those classes in order to achieve the objective (initialization of the stage)

### Participants
- **Facade** (`GameManager`)
    * knows which subsystem classes are responsible for a request.
    * delegates client requests to appropriate subsystem objects.
- **subsystem classes** (`MapCreator`, `MonsterSpawner`, `PlayerSpanwer`, etc.)
    * implement subsystem functionality.
    * handle work assigned by the Facade object.
    * have no knowledge of the facade;
        + that is, they keep no references to it.

### Sample Code
```c++
// this code exists for test purpose only
// this is not related to Facade pattern
enum class StageName { Forest, Mountain, Ocean };
struct PlayerInfo {
	// player data
};
struct Map {
	// map data
};


// Facade
class GameManager {
public:
	void initStage(const StageName &stageName, const PlayerInfo &playerInfo);
	// other members
};


// Subsystem Classes
class MapCreator {
public:
	MapCreator(const StageName &stageName) : curStage(stageName), curMap(nullptr) { 
		// do something
	}
	Map* createMap() {
		// do something
		std::cout << "the map has been created" << std::endl;

		return curMap;
	}
	// other members
private:
	StageName curStage;
	Map* curMap;
};

class MonsterSpawner {
public:
	MonsterSpawner(const StageName& stageName) : curStage(stageName) {
		// do something
	}
	void spawnMonsters(Map* targetMap) {
		// spawn monsters
		std::cout << "monsters have been spawned" << std::endl;
	}
	// other members
private:
	StageName curStage;
};

class PlayerSpawner {
public:
	PlayerSpawner(const PlayerInfo& playerInfo) : info(playerInfo) {
		// do something
	}
	void spawnPlayer(Map* targetMap) {
		// spawn player
		std::cout << "the player has been spawned" << std::endl;
	}
private:
	PlayerInfo info;
};

class HUD {
public:
	HUD(const PlayerInfo& inputPlayerData) : playerData(inputPlayerData) {
		// do something
	}
	void displayHUD() {
		// display current player's data
		std::cout << "the data is shown" << std::endl;
	}
private:
	PlayerInfo playerData;
};

class BGM {
public:
	BGM(const StageName& stageName) : curStage(stageName) {
		// do something
	}
	void playBGM() {
		// play the proper bgm based on the current stage
		std::cout << "the music is being played" << std::endl;
	}
private:
	StageName curStage;
};



void GameManager::initStage(const StageName& stageName, const PlayerInfo& playerInfo) {
	MapCreator mapCreator(stageName);
	MonsterSpawner monsterSpawner(stageName);
	BGM musicPlayer(stageName);

	HUD dataDisplayer(playerInfo);
	PlayerSpawner playerSpawner(playerInfo);


	Map* newMap = mapCreator.createMap();
	monsterSpawner.spawnMonsters(newMap);
	musicPlayer.playBGM();

	dataDisplayer.displayHUD();
	playerSpawner.spawnPlayer(newMap);
}



// some codes
GameManager gameManager;
PlayerInfo playerInfo;

gameManager.initStage(StageName::Ocean, playerInfo);

/*
print result
the map has been created
monsters have been spawned
the music is being played
the data is shown
the player has been spawned
*/
```

### Implementation
Consider the following issues when implementing a facade:
1. *Reducing client-subsystem coupling*
    * The coupling between clients and the subsystem can be reduced even further by making Facade an **abstract** class with concrete subclasses for different implementations of a subsystem.
        + Then clients can communicate with the subsystem through the interface of the abstract Facade class.
        + This abstract coupling keeps clients from knowing which implementation of a subsystem is used.
    * An **alternative** to subclassing is to **configure** a Facade object with different subsystem objects.
        + To customize the facade, simply replace one or more of its subsystem objects.
2. *Public versus private subsystem classes*
    * The public interface to a subsystem consists of classes that all clients can access
    * the private interface is just for subsystem extenders.
    * The Facade class is part of the public interface, of course, but it's not the only part.
        + Other subsystem classes are usually public as well.

### Related Patterns
- **Abstract Factory** can be used with Facade to provide an interface for creating subsystem objects in a subsystem-independent way.
    * Abstract Factory can also be used as an alternative to Facade to hide platform-specific classes.
- **Mediator** is similar to Facade in that it abstracts functionality of existing classes.
    * However, Mediator's purpose is to abstract arbitrary communication between colleague objects, often centralizing functionality that doesn't belong in any one of them.
    * A mediator's colleagues are aware of and communicate with the mediator instead of communicating with each other directly.
        + In contrast, a facade merely abstracts the interface to subsystem objects to make them easier to use; it doesn't define new functionality, and subsystem classes don't know about it.
- Usually only one Facade object is required. Thus Facade objects are often **Singleton**s.


## Consequences
The Facade pattern offers the following benefits:
1. It shields clients from subsystem components,
    * thereby reducing the number of objects that clients deal with and making the subsystem easier to use.
2. It promotes weak coupling between the subsystem and its clients.
    * Facades help layer a system and the dependencies between objects.
        + They can eliminate complex or circular dependencies.
        + This can be an important consequence when the client and the subsystem are implemented independently.
Reducing compilation dependencies is vital in large software systems. You want to save time by minimizing recompilation when subsystem classes change.
Reducing compilation dependencies with facades can limit the recompilation needed for a small change in an important subsystem. A façade can also simplify porting systems to other platforms, because it's less likely that building one subsystem requires building all others.
3. It doesn't prevent applications from using subsystem classes if they need to. Thus you can choose between ease of use and generality.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}