---
title: "Design Patterns : Interpreter"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, Interpreter]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-24
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Interpreter


## Problem

### Intent
Given a **language**, define a representation for its **grammar** along with an interpreter that uses the representation to **interpret** sentences in the language.

### Applicability
Use the **Interpreter** pattern when there is a language to interpret, and you can represent statements in the language as abstract syntax trees.
- The Interpreter pattern works best when
    * the grammar is **simple**.
        + For complex grammars, the class hierarchy for the grammar becomes large and unmanageable.
        + Tools such as parser generators are a better alternative in such cases.
        + They can interpret expressions without building abstract syntax trees, which can save space and possibly time.
    * **efficiency** is **not** a critical **concern**.
        + The most efficient interpreters are usually not implemented by interpreting parse trees directly but by first translating them into another form.
        + For example, regular expressions are often transformed into state machines.
        + But even then, the translator can be implemented by the Interpreter pattern, so the pattern is still applicable.


## Solution

### Motivation
Suppose that you want to implement a system which manipulates and evaluate `bool` expressions.
- The **problem** is that you need to implement the grammer rule as the programming langauge.
- A **solution** is to divide the expressions into 2 sections:
    * the one that has the meaning **by itself**
        + this is referred as **TerminalExpression**
        + ex: `true`
    * the one that has the meaning based on the **different value(s)**
        + this is referred as **NonterminalExpression**
        + ex: `true || false`, `!false`
    * and then, define the abstract base class which has the `interpret()` method that does the proper work based on the current derived class's job


### Participants
- **AbstractExpression** (`BooleanExpression`)
    * declares an abstract Interpret operation that is common to all nodes in the abstract syntax tree.
- **TerminalExpression** (`VariableExpression`, `ConstantExpression`)
    * implements an Interpret operation associated with terminal symbols in the grammar.
    * an instance is required for every terminal symbol in a sentence.
- **NonterminalExpression** (`AndExpression`, `OrExpression`, `NotExpression`)
    * one such class is required for every rule R := R1, R2, ... , Rn, in the grammar.
    * maintains instance variables of type AbstractExpression for each of the symbols R1 through Rn.
    * implements an Interpret operation for nonterminal symbols in the grammar.        
        + Interpret typically calls itself recursively on the variables representing R1 through Rn.
- **Context** (`Context`)
    * contains information that's global to the interpreter.
- **Client**
    * builds (or is given) an abstract syntax tree representing a particular sentence in the language that the grammar defines.
        + The abstract syntax tree is assembled from instances of the NonterminalExpression and TerminalExpression classes.
    * invokes the Interpret operation.

### Sample Code
```c++
// AbstractExpression
class Context;
class BooleanExpression {
public:
	virtual ~BooleanExpression() = default;
	virtual bool interpret(const Context&) = 0;
};

// TerminalExpression
class VariableExpression : public BooleanExpression {
	friend Context;
public:
	VariableExpression(const std::string & inputName) : name(inputName) { }
	bool interpret(const Context&) override;
private:
	std::string name;
};
class ConstantExpression : public BooleanExpression {
public:
	ConstantExpression(const bool inputValue) : value(inputValue) { }
	bool interpret(const Context&) override;
private:
	bool value;
};

// NonterminalExpression
class AndExpression : public BooleanExpression {
public:
	AndExpression(BooleanExpression* lhs, BooleanExpression* rhs) : leftOperand(lhs), rightOperand(rhs) { }
	bool interpret(const Context&) override;
private:
	BooleanExpression* leftOperand;
	BooleanExpression* rightOperand;
};
class OrExpression : public BooleanExpression {
public:
	OrExpression(BooleanExpression* lhs, BooleanExpression* rhs) : leftOperand(lhs), rightOperand(rhs) { }
	bool interpret(const Context&) override;
private:
	BooleanExpression* leftOperand;
	BooleanExpression* rightOperand;
};
class NotExpression : public BooleanExpression {
public:
	NotExpression(BooleanExpression* input) : operand(input) { }
	bool interpret(const Context&) override;
private:
	BooleanExpression* operand;
};


// Context
class Context {
public:
	bool lookUp(const VariableExpression& target) const {
		try {
			return data.at(target.name);
		}
		catch (std::out_of_range) {
			std::cerr << "\nThe given target (" << target.name << ") is not assigned to data. Default value(false) is returned." << std::endl;
			return false;
		}
	}
	void assign(const VariableExpression & target, const bool value) {
		data[target.name] = value;
	}
private:
	std::map<std::string, bool> data;
};

// Terminal
bool VariableExpression::interpret(const Context& context) {
	std::cout << context.lookUp(*this);
	return context.lookUp(*this);
}
bool ConstantExpression::interpret(const Context&) {
	std::cout << value;
	return value;
}

// Nonterminal
bool AndExpression::interpret(const Context& context) {
	std::cout << "AND( ";
	bool leftResult = leftOperand->interpret(context);
	std::cout << ", ";
	bool rightResult = rightOperand->interpret(context);
	std::cout << " )";

	return leftResult && rightResult;
}
bool OrExpression::interpret(const Context& context) {
	std::cout << "OR( ";
	bool leftResult = leftOperand->interpret(context);
	std::cout << ", ";
	bool rightResult = rightOperand->interpret(context);
	std::cout << " )";

	return leftResult || rightResult;
}
bool NotExpression::interpret(const Context& context) {
	std::cout << "NOT( ";
	bool innerResult = operand->interpret(context);
	std::cout << " )";

	return !innerResult;
}




// some codes
BooleanExpression* expression = nullptr;
Context context;

VariableExpression *x = new VariableExpression("X");
VariableExpression *y = new VariableExpression("Y");

expression = new OrExpression(
    new AndExpression(new ConstantExpression(true), x),
    new AndExpression(y, new NotExpression(x))
);


context.assign(*x, false);
context.assign(*y, true);


std::cout << std::boolalpha;
std::cout << "\nresult: " << expression->interpret(context) << std::endl;
/*
print result
OR( AND( true, false ), AND( true, NOT( false ) ) )
result: true
*/
```

### Implementation
The Interpreter and Composite patterns share many implementation issues.
- The following issues are specific to Interpreter:
1. *Creating the abstract syntax tree*
    * The Interpreter pattern doesn't explain how to create an abstract syntax tree.
        + In other words, it doesn't address parsing.
    * The abstract syntax tree can be created by a table-driven parser, by a hand-crafted (usually recursive descent) parser, or directly by the client.
2. *Defining the interpret operation*
    * You don't have to define the `interpret()` operation in the expression classes.
        + If it's common to create a new interpreter, then it's better to use the **Visitor** pattern to put `interpret()` in a separate "visitor" object.
    * For example, a grammar for a programming language will have many operations on abstract syntax trees, such as as type-checking, optimization, code generation, and so on.
        + It will be more likely to use a visitor to avoid defining these operations on every grammar class.
3. *Sharing terminal symbols with the Flyweight pattern*
    * Grammars whose sentences contain many occurrences of a terminal symbol might benefit from sharing a single **copy** of that symbol.
    * Grammars for computer programs are good examples
        + each program variable will appear in many places throughout the code.
    * Terminal nodes generally don't store information about their position in the abstract syntax tree.
        + Parent nodes pass them whatever context they need during interpretation. 
        + Hence there is a distinction between shared (intrinsic) state and passed-in (extrinsic) state, and the **Flyweight** pattern applies.

### Related Patterns
- **Composite**: The abstract syntax tree is an instance of the Composite pattern
- **Flyweight** shows how to share terminal symbols within the abstract syntax tree.
- **Iterator**: The interpreter can use an Iterator to traverse the structure.
- **Visitor** can be used to maintain the behavior in each node in the abstract syntax tree in one class.


## Consequences
The Interpreter pattern has the following benefits and liabilities:
1. *It's easy to change and extend the grammar*
    * Because the pattern uses classes to represent grammar rules, you can use inheritance to change or extend the grammar.
        + Existing expressions can be modified incrementally and new expressions can be defined as variations on old ones.
2. *Implementing the grammar is easy, too*
    * Classes defining nodes in the abstract syntax tree have similar implementations. 
        + These classes are easy to write, and often their generation can be automated with a compiler or parser generator.
3. *Complex grammars are hard to maintain*
    * The Interpreter pattern defines at least one class for every rule in the grammar (grammar rules defined using BNF may require multiple classes).
        + Hence grammars containing many rules can be hard to manage and maintain. 
    * Other design patterns can be applied to mitigate the problem (see **Implementation** section).
        + But when the grammar is very complex, other techniques such as parser or compiler generators are more appropriate.
4. *Adding new ways to interpret expressions*
    * The Interpreter pattern makes it easier to evaluate an expression in a new way. 
        + For example, you can support pretty printing or type-checking an expression by defining a new operation on the expression classes.
    * If you keep creating new ways of interpreting an expression, then consider using the Visitor pattern to avoid changing the grammar classes.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}