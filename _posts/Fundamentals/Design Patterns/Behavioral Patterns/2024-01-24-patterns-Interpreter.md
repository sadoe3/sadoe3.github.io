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
Given a language, define a representation for its grammar along with an interpreter that uses the representation to interpret sentences in the language.

### Applicability
Use the **Interpreter** pattern when there is a language to interpret, and you can represent statements in the language as abstract syntax trees.
- The Interpreter pattern works best when
    * the grammar is simple.
        + For complex grammars, the class hierarchy for the grammar becomes large and unmanageable.
        + Tools such as parser generators are a better alternative in such cases.
        + They can interpret expressions without building abstract syntax trees, which can save space and possibly time.
    * efficiency is not a critical concern.
        + The most efficient interpreters are usually not implemented by interpreting parse trees directly but by first translating them into another form.
        + For example, regular expressions are often transformed into state machines.
        + But even then, the translator can be implemented by the Interpreter pattern, so the pattern is still applicable.


## Solution

### Motivation
Suppose that ~
- The **problem** is that ~
- A **solution** is to ~

### Participants
- **AbstractExpression** (`RegularExpression`)
    * declares an abstract Interpret operation that is common to all nodes in the abstract syntax tree.
- **TerminalExpression** (`LiteralExpression`)
    * implements an Interpret operation associated with terminal symbols in the grammar.
    * an instance is required for every terminal symbol in a sentence.
- **NonterminalExpression** (`AlternationExpression`, `RepetitionExpression`, `SequenceExpressions`)
    * one such class is required for every rule R := R, R,... R, in the grammar.
    * maintains instance variables of type AbstractExpression for each of the symbols R, through R,.
    * implements an Interpret operation for nonterminal symbols in the grammar.        
        + Interpret typically calls itself recursively on the variables representing R, through R,
- **Context**
    * contains information that's global to the interpreter.
- **Client**
    * builds (or is given) an abstract syntax tree representing a particular sentence in the language that the grammar defines. The abstract syntax tree is assembled from instances of the NonterminalExpression and TerminalExpression classes.
    * invokes the Interpret operation.

### Sample Code
```c++

```

### Implementation
The Interpreter and Composite patterns share many implementation issues.
- The following issues are specific to Interpreter:
1. *Creating the abstract syntax tree*
    * The Interpreter pattern doesn't explain how to create an abstract syntax tree. In other words, it doesn't address parsing. The abstract syntax tree can be created by a table-driven parser, by a hand-crafted (usually recursive descent) parser, or directly by the client.
2. *Defining the Interpret operation*
    * You don't have to define the Interpret operation in the expression classes. If it's common to create a new interpreter, then it's better to use the Visitor pattern to put Interpret in a separate "visitor" object. For example, a grammar for a programming language will have many operations on abstract syntax trees, such as as type-checking, optimization, code generation, and so on.
It will be more likely to use a visitor to avoid defining these operations on every grammar class.
3. *Sharing terminal symbols with the Flyweight pattern*
    * Grammars whose sentences contain many occurrences of a terminal symbol might benefit from sharing a single copy of that symbol. Grammars for computer programs are good examples-each program variable will appear in many places throughout the code. In the Motivation example, a sentence can have the terminal symbol dog (modeled by the LiteralExpression class) appearing many times.
    * Terminal nodes generally don't store information about their position in the abstract syntax tree. Parent nodes pass them whatever context they need during interpretation. Hence there is a distinction between shared (intrinsic) state and passed-in (extrinsic) state, and the Flyweight pattern applies.
    * For example, each instance of LiteralExpression for dog receives a context containing the substring matched so far. And every such LiteralExpression does the same thing in its Interpret operation-it checks whether the next part of the input contains a dog—no matter where the instance appears in the tree.

### Related Patterns
- **Composite**: The abstract syntax tree is an instance of the Composite pattern
- **Flyweight** shows how to share terminal symbols within the abstract syntax tree.
- **Iterator**: The interpreter can use an Iterator to traverse the structure.
- **Visitor** can be used to maintain the behavior in each node in the abstract syntax tree in one class.


## Consequences
The Interpreter pattern has the following benefits and liabilities:
1. *It's easy to change and extend the grammar*
    * Because the pattern uses classes to represent grammar rules, you can use inheritance to change or extend the grammar.
Existing expressions can be modified incrementally and new expressions can be defined as variations on old ones.
2. *Implementing the grammar is easy, too*
    * Classes defining nodes in the abstract syntax tree have similar implementations. These classes are easy to write, and often their generation can be automated with a compiler or parser generator.
3. *Complex grammars are hard to maintain*
    * The Interpreter pattern defines at least one class for every rule in the grammar (grammar rules defined using BNF may require multiple classes). Hence grammars containing many rules can be hard to manage and maintain. Other design patterns can be applied to mitigate the problem (see Implementation). But when the grammar is very complex, other techniques such as parser or compiler generators are more appropriate.
4. *Adding new ways to interpret expressions*
    * The Interpreter pattern makes it easier to evaluate an expression in a new way. For example, you can support pretty printing or type-checking an expression by defining a new operation on the expression classes. If you keep creating new ways of interpreting an expres-sion, then consider using the Visitor pattern to avoid changing the grammar classes.    

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}