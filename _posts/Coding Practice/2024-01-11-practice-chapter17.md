---
title: "Clean Code : Chapter 17"

categories:
    - coding-practice

tags:
    - [Clean Code, Coding Practice, Refactoring, Heuristics]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-11
---

# Smells and Heuristics	

> 이 포스트는 Clean Code(1st Edition)를 바탕으로 작성되었습니다.

## Refactoring
Writing **clean** code in **one pass** is **not** the objective you need to **achieve**
- your **objective** should be **refactoring** your code **continually** so that the code **naturally** becomes **clean**
- it doesn't have to be clean at first
    * in general, to write clean code
        + you must **first** write **dirty** code, and **then clean** it


## General

### G11: Inconsistency
Be careful with the **conventions** you choose, and once chosen, be careful to **continue** to follow them
- simple **consistency** like this, when reliably applied, can make code much **easier to read and modify**

### G24: Follow Standard Conventions
Every team should **follow** a coding **standard** based on common **industry norm**
A- this coding standard should specify:
	* **where** to declare data members
	* **how** to name classes, methods, and variables
	* ** where** to put braces; and so on
A- the team should **not need a document** to describe these conventions because their **code** provides the **examples**

### G12: Clutter
Keep your source files **clean**, well organized, and **free** of **clutter**
- clutter should be **removed** and clutter can be
	* variables that are **not used**
	* functions that are **never called**
	* comments that add **no information**

### G14: Feature Envy
The **methods** of a class should be interested in the variables and functions of the **class they belong to**, and **not** the variables and functions of **other classes** 
- otherwise, **Feature Envy** would **expose** the **internals** of one class to another
	* and this would **increase the dependency** which we don’t desire

### G15: Selector Arguments
Containing a function which acts **differently** based on the **argument** is probably a **sign** that it takes **multiple purposes**
- when this happens, **decompose** it so that each new function would contain **single** purpose

### G16: Obscure Intent
**High readability** is one of the virtues which the **clean** code has
- we want code to be as expressive as possible
	* run-on expressions, Hungarian notation, and magic numbers all **obscure** the author’s intent
	* therefore, they should be **avoided**

### G25: Replace Magic Numbers with Named Constants
A **magic number** is a **unique value** with **unexplained** meaning
```c++
// bad
const int TWO = 2;
double circumference = 12.4 * 3.14 * TWO;

printEmployeeCard(32);


// clean
double circumference = radius * std::numbers::pi * 2;

constexpr int EMPLOYEE_ID = 3;
printEmployeeCard(EMPLOYEE_ID);
```
- it’s worth noting that this heuristic does **not** say replacing all numeric literals with named constants
	*  if the numeric literals are **self-describing** enough, then it does **not need any named constants**

### G33: Encapsulate Boundary Conditions
A **boundary condition** should be **encapsulated** within a **named variable or constant**
```c++
// bad
if(level + 1 < tags.length) {
	parts = new Parse(body, tags, level + 1, offset + endTag);
	body = nullptr;
}

// clean
const nextLevel = level + 1;
if(nextLevel < tags.length) {
	parts = new Parse(body, tags, nextLevel, offset + endTag);
	body = nullptr;
}
```

### G20: Function Names Should Say What they Do
**Ambiguous** names of functions **decrease the readability** of the code
```c++
// bad
Date newDate = date.add(5);

// clean
Date newDate = date.getDateDaysLater(5);
```
- if you **have to look at the implementation** (or documentation) of the function **to know what it does**
	* then you should work to 
		+ find a **better name** or
		+ **rearrange the functionality** so that it can be placed in functions with better names

### G21: Understand the Algorithm
**Before** you consider yourself to be **done** with a function, make sure you **understand** how it works with the **control flow**
- it is **not** good **enough** that it **passes** all the tests
	* you must **know why** it passes them






### 22(easy to use correctly + 27 + 31)
**Not done yet**






### G23: Prefer Polymorphism to `If/Else` or `Switch/Case`
**Most** people use `switch` statements because it’s the obvious **brute force** solution, **not** because it’s the **right** solution for the situation
- hence, this heuristic is here to **remind** us to consider **polymorphism** before using a `switch`

### G28: Encapsulate Conditionals
Replace **complex conditions** with a **simple function** which checks the conditions instead
```c++
// bad
if( timer.hasExpired() && !timer.isRecurrent() )
	;

// clean
inline bool shouldBeDeleted(const Timer &timer) {
	return timer.hasExpired() && !timer.isRecurrent();
}
if(shouldBeDeleted(timer))
	;
```
- this kind of function is recommended to be `inline`d

### G29: Avoid Negative Conditionals
We need to avoid **double negatives**
```c++
// bad
if(!buffer.shouldNotCompact())
	;

// clean
if(buffer.shouldCompact())
	;
```
- **double negatives** is the combination of a **condition** contains `!(NOT operator)` **and** the **name** of an object or a method contains a **negative word**
	* when this happens, try to **eliminate** the `!(NOT operator)` and the **negative word**
	* so that we can **increase the readability** of the code




### 34, 35
**not done yet**





### G36: Avoid Transitive Navigation
In general we do **not** want a single class to **know** much about its **collaborators**
```c++
// bad
a.getB().getC().doSomething();

// clean
myCollaborator.doSomething();
```
- if `A` collaborates with `B`, and `B` collaborates with `C`
	* then, we do **not** want classes that use `A` to know about `C`
    * make sure that classes know **only** about their **immediate collaborators** and
        + do **not know** the navigation map of the **whole system**
- this is sometimes called the **Law of Demeter**



## Names

### N1: Choose Descriptive Names
Again, the **readability** matters. You need to take the time to choose **descriptive** names wisely and keep them **relavant**
```c++
// bad
int i = 3;
arr[i].a();

// clean
constexpr unsigned targetPage = 2;
pages[targetPage].print();
```

### N2: Choose Names at the Appropriate Level of Abstraction


### N3: Use Standard Nomenclature Where Possible

### N4: Unambiguous Names




## Tests

### T1: Insufficient Tests
1,3~7, 9


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}