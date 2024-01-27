---
title: "Design Patterns : Command"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, Command]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-24
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Command

### Also Known As
Action, Transaction


## Problem

### Intent
Encapsulate a **request** as an **object**, thereby letting you **parameterize** clients with different requests, queue or log requests, and support **undoable** operations.

### Applicability
Use the **Command** pattern when you want to
- **parameterize** objects by an **action** to perform.
    * You can express such parameterization in a procedural language with a callback function
        + that is, a function that's registered somewhere to be called at a later point.
    * Commands are an object-oriented replacement for callbacks.
- specify, **queue**, and execute **requests** at different times.
    * A Command object can have a **lifetime independent** of the original request.
    * If the receiver of a request can be represented in an address space-independent way
        + then you can transfer a command object for the request to a different process and fulfill the request there.
- support **undo**
    * The Command's Execute operation can store state for reversing its effects in the command itself.
    * The Command interface must have an added Unexecute operation that reverses the effects of a previous call to Execute.
    * Executed commands are stored in a history list.
    * Unlimited-level undo and redo is achieved by traversing this list backwards and forwards calling Unexecute and Execute, respectively.
- support logging changes so that they can be reapplied in case of a system crash
    * By augmenting the Command interface with load and store operations
        + you can keep a persistent log of changes.
    * Recovering from a crash involves reloading logged commands from disk and reexecuting them with the Execute operation.


## Solution

### Motivation
Suppose that you want to implement an application which has 4 buttons with the random number to control current number. The button can add to or subtract from the current number with its own number.
- The **problem** is that you want to **decouple** the button object from the object which actually adds or subtracts.
    + Moreover, you want to implement a **undo** and **redo** features.
- A **solution** is to define a class which handles the **history** of commands so that you can undo or redo easily.

### Participants
- **Command** (`Command`)
    * declares an interface for executing an operation.
- **ConcreteCommand** (`CommandAdd`, `CommandSubtract`, `CommandUndo`, etc.)
    * defines a binding between a Receiver object and an action.
    * implements Execute by invoking the corresponding operation(s) on Receiver.
- **Client** 
    * creates a ConcreteCommand object and sets its receiver.
- **Invoker** (`Button` objects)
    * asks the command to carry out the request.
        + it's worth noting that Invoker objects can be implemented through inheritance or composition
- **Receiver** (`NumberPlayer`, `ApplicationManager`)
    * knows how to perform the operations associated with carrying out a request.
    * Any class may serve as a Receiver.

### Sample Code
```c++
// this code exists for test purpose only
// this is not related to Command pattern
enum class ButtonType { TopRight, TopLeft, BottomLeft, BottomRight, Undo, Redo, Exit , LastButtonType };
// Only Add and Subtract operations are undoable
enum class CommandType { Add, Subtract, LastCommandType };
std::string getButtonName(const ButtonType& type) {
	switch (type) {
	case ButtonType::TopRight:
		return "TopRight";
	case ButtonType::TopLeft:
		return "TopLeft";
	case ButtonType::BottomLeft:
		return "BottomLeft";
	case ButtonType::BottomRight:
		return "BottomRight";
	case ButtonType::Undo:
		return "Undo";
	case ButtonType::Redo:
		return "Redo";
	case ButtonType::Exit:
		return "Exit";
	}
	return "LastButtonType";
}



// For redo and undo
// history data member takes the pair of the command and the button number
class Command;
class CommandHistory {
	friend Command;
public:
	static void addExecutedCommandByButtonClick(const CommandType&, const int &);
private:
	static std::stack<std::pair<CommandType, int>> historyUndo;
	static std::stack<std::pair<CommandType, int>> historyRedo;
};
std::stack<std::pair<CommandType, int>> CommandHistory::historyUndo;
std::stack<std::pair<CommandType, int>> CommandHistory::historyRedo;
void CommandHistory::addExecutedCommandByButtonClick(const CommandType& commandType, const int& buttonNumber) {
	while (historyRedo.empty() == false)
		historyRedo.pop();
	historyUndo.emplace(commandType, buttonNumber);
}



// Command
class Command {
public:
	virtual ~Command() = default;
	virtual void execute() = 0;
protected:
	int getRecentButtonNumber() {
		if (CommandHistory::historyUndo.empty() == false)
			return CommandHistory::historyUndo.top().second;
		return 0;
	}
	void setRecentCommandType(const CommandType& inputCommandType) {
		if(CommandHistory::historyUndo.empty() == false)
			CommandHistory::historyUndo.top().first = inputCommandType;
	}
	std::pair<CommandType, int> popRecentCommand() {
		std::pair<CommandType, int> recentCommand = { CommandType::LastCommandType, 0 };
		if (CommandHistory::historyUndo.empty() == false) {
			recentCommand = CommandHistory::historyUndo.top();
			CommandHistory::historyRedo.push(recentCommand);
			CommandHistory::historyUndo.pop();
		}
		return recentCommand;
	}
	std::pair<CommandType, int> popRecentRedoCommand() {
		std::pair<CommandType, int> recentRedoCommand = { CommandType::LastCommandType, 0 };
		if (CommandHistory::historyRedo.empty() == false) {
			recentRedoCommand = CommandHistory::historyRedo.top();
			CommandHistory::historyRedo.pop();
		}
		return recentRedoCommand;
	}
	void addExecutedCommandByRedo(const CommandType& commandType, const int& buttonNumber) {
		CommandHistory::historyUndo.emplace(commandType, buttonNumber);
	}
};


// Receiver
class NumberPlayer {
public:
	NumberPlayer(const int & initialNumber) : number(initialNumber) { }
	void addNumber(const int& rhs) {
		number += rhs;
	}
	void subtractNumber(const int& rhs) {
		number -= rhs;
	}
	int getNumber() {
		return number;
	}
private:
	int number;
};

struct ApplicationManager {
	bool isEnd = false;
};



// ConcreteCommand
class CommandAdd : public Command {
public:
	CommandAdd(NumberPlayer &inputNumberPlayer) : numberPlayer(inputNumberPlayer) { }
	void execute() override {
		numberPlayer.addNumber(Command::getRecentButtonNumber());
		Command::setRecentCommandType(CommandType::Add);
	}
private:
	NumberPlayer& numberPlayer;
};
class CommandSubtract : public Command {
public:
	CommandSubtract(NumberPlayer& inputNumberPlayer) : numberPlayer(inputNumberPlayer) { }
	void execute() override {
		numberPlayer.subtractNumber(Command::getRecentButtonNumber());
		Command::setRecentCommandType(CommandType::Subtract);
	}
private:
	NumberPlayer& numberPlayer;
};
class CommandUndo : public Command {
public:
	CommandUndo(NumberPlayer& inputNumberPlayer) : numberPlayer(inputNumberPlayer) { }
	void execute() override {
		std::pair<CommandType, int> recentCommand = Command::popRecentCommand();
		switch (recentCommand.first) {
		case CommandType::Add:
			numberPlayer.subtractNumber(recentCommand.second);
			break;
		case CommandType::Subtract:
			numberPlayer.addNumber(recentCommand.second);
			break;
		default:
			std::cout << "there's nothing undoable" << std::endl;
		}
	}
private:
	NumberPlayer& numberPlayer;
};
class CommandRedo : public Command {
public:
	CommandRedo(NumberPlayer& inputNumberPlayer) : numberPlayer(inputNumberPlayer) { }
	void execute() override {
		std::pair<CommandType, int> recentCommand = Command::popRecentRedoCommand();
		switch (recentCommand.first) {
		case CommandType::Add:
			numberPlayer.addNumber(recentCommand.second);
			break;
		case CommandType::Subtract:
			numberPlayer.subtractNumber(recentCommand.second);
			break;
		default:
			std::cout << "there's nothing redoable" << std::endl;
			return;
		}
		Command::addExecutedCommandByRedo(recentCommand.first, recentCommand.second);
	}
private:
	NumberPlayer& numberPlayer;
};
class CommandExit : public Command {
public:
	CommandExit(ApplicationManager& inputApplicationManager) : applicationManager(inputApplicationManager) { }
	void execute() override {
		applicationManager.isEnd = true;
	}
private:
	ApplicationManager& applicationManager;
};



// Invoker
class Button {
public:
	Button(const ButtonType &inputType, const int & inputNumber, Command* inputCommand) : BUTTON_TYPE(inputType), BUTTON_NUMBER(inputNumber), command(inputCommand) { }
	~Button() {
		delete command;
	}
	void click() {
		std::cout << getButtonName(BUTTON_TYPE) << " Button is clicked" << std::endl;
		if(static_cast<int>(BUTTON_TYPE) < static_cast<int>(ButtonType::Undo))
			CommandHistory::addExecutedCommandByButtonClick(CommandType::LastCommandType, BUTTON_NUMBER);
		command->execute();
	}
private:
	const ButtonType BUTTON_TYPE;
	const int BUTTON_NUMBER;
	Command* command;
};




// some codes
const int MAX_BUTTON_NUMBER = 30;
const int MIN_TARGET_NUMBER = -40;
const int MAX_TARGET_NUMBER = 40;
const int MIN_PLAYER_NUMBER = -10;
const int MAX_PLAYER_NUMBER = 10;

std::default_random_engine randomEngine;
std::uniform_int_distribution<int> randomButtonNumber(0, MAX_BUTTON_NUMBER);
std::uniform_int_distribution<int> randomTargetNumber(MIN_TARGET_NUMBER, MAX_TARGET_NUMBER);
std::uniform_int_distribution<int> randomPlayerNumber(MIN_PLAYER_NUMBER, MAX_PLAYER_NUMBER);
std::uniform_int_distribution<int> randomIsAdd(0, 1);


NumberPlayer player(randomPlayerNumber(randomEngine));
ApplicationManager applicationManager;

std::vector<Button*> buttons;
Command* newCommand = nullptr;
for (int currentButtonType = 0, endButtonType = static_cast<int>(ButtonType::Undo); currentButtonType < endButtonType; currentButtonType++) {
    if (randomIsAdd(randomEngine))
        newCommand = new CommandAdd(player);
    else
        newCommand = new CommandSubtract(player);

    buttons.push_back(new Button(static_cast<ButtonType>(currentButtonType), randomButtonNumber(randomEngine), newCommand));
}
buttons.push_back(new Button(ButtonType::Undo, 0, new CommandUndo(player)));
buttons.push_back(new Button(ButtonType::Redo, 0, new CommandRedo(player)));
buttons.push_back(new Button(ButtonType::Exit, 0, new CommandExit(applicationManager)));


int userInput = 0;
while (applicationManager.isEnd == false) {
    std::cout << "\nchoose your button ( 0(TopRight), 1(TopLeft), 2(BottomLeft), 3(BottomRight), 4(Undo), 5(Redo), 6(Exit) ) : ";
    std::cin >> userInput;
    if (userInput < 0 || userInput > 6)
        std::cout << "choose between 0 ~ 6" << std::endl;
    else {
        buttons[userInput]->click();
        std::cout << "your number: " << player.getNumber() << std::endl;
    }
}


for (auto currentButton : buttons)
    delete currentButton;

/*
print result
choose your button ( 0(TopRight), 1(TopLeft), 2(BottomLeft), 3(BottomRight), 4(Undo), 5(Redo), 6(Exit) ) : 0
TopRight Button is clicked
your number: -31

choose your button ( 0(TopRight), 1(TopLeft), 2(BottomLeft), 3(BottomRight), 4(Undo), 5(Redo), 6(Exit) ) : 4
Undo Button is clicked
your number: -2

choose your button ( 0(TopRight), 1(TopLeft), 2(BottomLeft), 3(BottomRight), 4(Undo), 5(Redo), 6(Exit) ) : 4
Undo Button is clicked
there's nothing undoable
your number: -2

choose your button ( 0(TopRight), 1(TopLeft), 2(BottomLeft), 3(BottomRight), 4(Undo), 5(Redo), 6(Exit) ) :
*/
```

### Implementation
Consider the following issues when implementing the Command pattern:
1. *How intelligent should a command be?*
    * A command can have a wide range of abilities.
        + At one extreme it merely defines a binding between a receiver and the actions that carry out the request.
        + At the other extreme it implements everything itself without delegating to a receiver at all.
            - The latter extreme is useful when you want to define commands that are independent of existing classes, when no suitable receiver exists, or when a command knows its receiver implicitly.
            - For example, a command that creates another application window may be just as capable of creating the window as any other object.
        + Somewhere in between these extremes are commands that have enough knowledge to find their receiver dynamically.
2. *Supporting undo and redo*
    * Commands can support undo and redo capabilities if they provide a way, to reverse their execution (e.g., an Unexecute or Undo operation). A ConcreteCommand class might need to store additional state to do so.
    * This state can include
        + the Receiver object, which actually carries out operations in response to the request,
        + the arguments to the operation performed on the receiver, and
        + any original values in the receiver that can change as a result of handling the request. The receiver must provide operations that let the command return the receiver to its prior state.
    * To support one level of undo, an application needs to store only the command that was executed last.
    * For multiple-level undo and redo, the application needs a **history list** of commands that have been executed, where the maximum length of the list determines the number of undo/redo levels.
        + The history list stores sequences of commands that have been executed.
        + Traversing backward through the list and reverse-executing commands cancels their effect;
            - traversing forward and executing commands reexecutes them.
    * An undoable command might have to be copied before it can be placed on the history list.
        + That's because the command object that carried out the original request will perform other requests at later times.
        + Copying is required to distinguish different invocations of the same command if its state can vary across invocations.
        + Commands that must be copied before being placed on the history list act as prototypes (see [**Prototype**](https://sadoe3.github.io/design-patterns/patterns-Prototype/)).
3. *Avoiding error accumulation in the undo process*
    * Errors can accumulate as commands are executed, unexecuted, and reexecuted repeatedly so that an application's state eventually diverges from original values.
        + It may be necessary therefore to store more information in the command to ensure that objects are restored to their original state.
    * The **Memento** pattern can be applied to give the command access to this information without exposing the internals of other objects.
4. *Using C++ templates*
    * For commands that
        + (1) aren't undoable and
        + (2) don't require arguments,
    * we can use C++ templates to avoid creating a Command subclass for every kind of action and receiver.

### Related Patterns
- A **Composite** can be used to implement MacroCommands.
- A **Memento** can keep state the command requires to undo its effect.
- A command that must be copied before being placed on the history list acts as a **Prototype**.


## Consequences
The Command pattern has the following consequences:
1. Command **decouples** the object that invokes the operation from the one that knows how to perform it.
2. You can assemble commands into a composite command. 
    * In general, composite commands are an instance of the **Composite** pattern.


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}