---
title: "Design Patterns : Memento"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, Memento]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-26
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Memento

### Also Known As
Token


## Problem

### Intent
**Without violating encapsulation**, capture and **externalize** an object's **internal state** so that the object can be restored to this state later.

### Applicability
Use the **Memento** pattern when
- a snapshot of (some portion of) an object's state must be **saved** so that it can be **restored** to that state later, and
- a direct interface to obtaining the state would expose implementation details and break the object's encapsulation.


## Solution

### Motivation
Suppose that you want to implement a simple program which takes the user input as an integral number to change player's current position, and prints current position.
- The **problem** is that you want to implement **undo** and **redo** features.
- A **solution** is to create and store the object which knows the internal data (position in this case) before changing player's value in a certain location.

### Participants
- **Memento** (`MementoPlayer`)
    * stores internal state of the Originator object.
        + The memento may store as much or as little of the originator's internal state as necessary at its originator's discretion.
    * protects against access by objects other than the originator.
        + Mementos have effectively two interfaces.
            - Caretaker sees a narrow interface to the Memento
                * it can only pass the memento to other objects.
            - Originator, in contrast, sees a wide interface, one that lets it access all the data necessary to restore itself to its previous state.
        + Ideally, only the originator that produced the memento would be permitted to access the memento's internal state.
- **Originator** (`OriginatorPlayer`)
    * creates a memento containing a snapshot of its current internal state.
    * uses the memento to restore its internal state.
        - it's worth noting that Originator can be a **Singleton**
- **Caretaker** (`CaretakerPlayer`)
    * is responsible for the memento's safekeeping.
    * never operates on or examines the contents of a memento.

### Sample Code
```c++
class MementoPlayer;
// Originator
// this doesn't have to be a Singleton
class OriginatorPlayer {
public:
	static OriginatorPlayer* getInstance();

	MementoPlayer* movePlayer(const int&);
	void restorePlayer(const MementoPlayer *);
protected:
	OriginatorPlayer() : positionX(0) { }
private:
	static OriginatorPlayer* instance;

	MementoPlayer* createMemento();
	int positionX;
	void printCurrentPosition() {
		std::cout << "current position : " << positionX << std::endl;
	}
};
OriginatorPlayer* OriginatorPlayer::instance = nullptr;
OriginatorPlayer* OriginatorPlayer::getInstance() {
	if (OriginatorPlayer::instance == nullptr) {
		OriginatorPlayer::instance = new OriginatorPlayer();
	}
	return OriginatorPlayer::instance;
}


// Memento
class MementoPlayer {
	friend OriginatorPlayer;
public:
	MementoPlayer(const int & inputX) : positionX(inputX) { }
	MementoPlayer() = delete;
private:
	int positionX;
};

MementoPlayer* OriginatorPlayer::movePlayer(const int& delta) {
	auto previousMemento = createMemento();
	positionX += delta;

	printCurrentPosition(); 

	return previousMemento;
}
void OriginatorPlayer::restorePlayer(const MementoPlayer* backUp) {
	positionX = backUp->positionX;

	printCurrentPosition();
}
MementoPlayer* OriginatorPlayer::createMemento() {
	return new MementoPlayer(positionX);
}


// Caretaker
class CaretakerPlayer {
public:
	CaretakerPlayer() : positions(), currentAvailableIndex(0), currentUndoIndex(-1) { }
	~CaretakerPlayer() {
		for (auto currentMemento : positions) {
			delete currentMemento.second;
		}
	}
	void movePlayer(const int& delta);
	void undoMove();
	void redoMove();
private:
	std::map<int, MementoPlayer*> positions;
	int currentAvailableIndex;
	int currentUndoIndex;
};
void CaretakerPlayer::movePlayer(const int& delta) {
	if (currentUndoIndex + 1 != currentAvailableIndex) {
		for (int startIndex = currentUndoIndex + 1; startIndex < currentAvailableIndex; startIndex++) {
			delete positions[startIndex];
			positions[startIndex] = nullptr;
		}
		currentAvailableIndex = currentUndoIndex + 1;
	}
	positions[currentAvailableIndex] = OriginatorPlayer::getInstance()->movePlayer(delta);
	currentUndoIndex = currentAvailableIndex;
	currentAvailableIndex++;

}
void CaretakerPlayer::undoMove() {
	if (currentUndoIndex > -1) {
		if(currentUndoIndex == 0 && positions.empty())
			std::cout << "there's nothing undoable" << std::endl;
		else {
			if (currentUndoIndex + 1 == currentAvailableIndex) {
				movePlayer(0);
				--currentUndoIndex;
				undoMove();
			}
			else {
				OriginatorPlayer::getInstance()->restorePlayer(positions[currentUndoIndex]);
				currentUndoIndex--;
			}
		}
	}
	else
		std::cout << "there's nothing undoable" << std::endl;
}
void CaretakerPlayer::redoMove() {
	currentUndoIndex++;
	if (currentUndoIndex+1 == currentAvailableIndex) {
		std::cout << "there's nothing redoable" << std::endl;
		currentUndoIndex--;
	}
	else 
		OriginatorPlayer::getInstance()->restorePlayer(positions[currentUndoIndex+1]);
}



// some codes
CaretakerPlayer player;
	
int userInput;
bool isDone = false;
while (!isDone) {
    std::cout << "Choose your action (0: move, 1: undo, 2: redo, 3: exit) : ";
    std::cin >> userInput;
    switch (userInput) {
    case 0:
        std::cout << "Enter delta value : ";
        std::cin >> userInput;
        player.movePlayer(userInput);
        break;
    case 1:
        player.undoMove();
        break;
    case 2:
        player.redoMove();
        break;
    case 3:
        isDone = true;
        break;
    default:
        std::cout << "enter proper input" << std::endl;
    }
}

/*
print result
Choose your action (0: move, 1: undo, 2: redo, 3: exit) : 0
Enter delta value : 1
current position : 1
Choose your action (0: move, 1: undo, 2: redo, 3: exit) : 0
Enter delta value : 10
current position : 11
Choose your action (0: move, 1: undo, 2: redo, 3: exit) : 0
Enter delta value : 100
current position : 111
Choose your action (0: move, 1: undo, 2: redo, 3: exit) : 1
current position : 111
current position : 11
Choose your action (0: move, 1: undo, 2: redo, 3: exit) : 1
current position : 1
Choose your action (0: move, 1: undo, 2: redo, 3: exit) : 1
current position : 0
Choose your action (0: move, 1: undo, 2: redo, 3: exit) : 1
there's nothing undoable
Choose your action (0: move, 1: undo, 2: redo, 3: exit) : 2
current position : 1
Choose your action (0: move, 1: undo, 2: redo, 3: exit) : 2
current position : 11
Choose your action (0: move, 1: undo, 2: redo, 3: exit) : 2
current position : 111
Choose your action (0: move, 1: undo, 2: redo, 3: exit) : 2
there's nothing redoable
Choose your action (0: move, 1: undo, 2: redo, 3: exit) : 3
*/
```

### Implementation
Here are two issues to consider when implementing the Memento pattern:
1. *Language support*
    * Mementos have two interfaces: a wide one for originators and a narrow one for other objects.
    * Ideally the implementation language will support two levels of static protection.
        + C++ lets you do this by making the Originator a `friend` of Memento and making Memento's wide interface private
        + Only the narrow interface should be declared `public`.
    * For example:
        ```c++
        class State;

        class Originator {
        public:
            Memento* createMemento();
            void setMemento(const Memento*);
            // ...
        private:
            State* state;   // internal data structures
            // ...
        };

        class Memento {
        public:
            // narrow public interface
            virtual Memento();
        private:
            // private members accessible only to Originator
            friend class Originator;
            Memento();

            void setState(State*);
            State* getState();
            // ...
            State* state;
            // ...
        };
        ```
2. *Storing incremental changes*
    * When mementos get created and passed back to their originator in a predictable sequence,
        + then Memento can save just the incremental change to the originator's internal state.
    * For example, **undoable** commands in a history list can use mementos to ensure that commands are restored to their exact state when they're undone (see **Command**).
        + The history list defines a specific order in which commands can be undone and redone.
        + That means mementos can store just the incremental change that a command makes rather than the full state of every object they affect.

### Related Patterns
- **Command**: Commands can use mementos to maintain state for undoable operations.
- **Iterator**: Mementos can be used for iteration as described earlier.


## Consequences
The Memento pattern has several consequences:
1. *Preserving encapsulation boundaries*
    * Memento **avoids exposing information** that only an originator should manage but that must be stored nevertheless outside the originator.
    * The pattern shields other objects from potentially complex Originator internals, thereby preserving encapsulation boundaries.
2. *Using mementos might be expensive*
    * Mementos might incur considerable **overhead**
        + if Originator must **copy large** amounts of information to store in the memento or
        + if clients create and return mementos to the originator often enough. 
    * Unless encapsulating and restoring Originator state is cheap, the pattern might not be appropriate.
    * See the discussion of incrementality in the **Implementation** section.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}