---
title: "Design Patterns : Template Method"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, Template Method]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-28
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Template Method


## Problem

### Intent
Define the **skeleton of an algorithm** in an operation, **deferring some steps to subclasses**. Template Method lets **subclasses redefine certain steps** of an algorithm without changing the algorithm's structure.

### Applicability
The **Template Method** pattern should be used
- to implement the invariant parts of an algorithm once and leave it up to subclasses to implement the behavior that can vary.
- when common behavior among subclasses should be factored and localized in a common class to avoid code duplication.
    * You first identify the differences in the existing code and then separate the differences into new operations.
    * Finally, you replace the differing code with a template method that calls one of these new operations.
- to control subclasses extensions.
    * You can define a template method that calls "hook" operations (see Consequences) at specific points, thereby permitting extensions only at those points.


## Solution

### Motivation
Suppose that you want to implement a simple program which has 2 speakers of different languages.
- The **problem** is that they speak in the same manner, the only difference is just the language.
- A **solution** is to define the common steps of speaking, and then make the derived classes override the language part only.

### Participants
- **AbstractClass** (`Speaker`)
    * defines abstract **primitive operations** that concrete subclasses define to implement steps of an algorithm.
    * implements a template method defining the skeleton of an algorithm.
        + The template method calls primitive operations as well as operations defined in AbstractClass or those of other objects.
- **ConcreteClass** (`EnglishSpeaker`, `KoreanSpeaker`)
    * implements the primitive operations to carry out subclass-specific steps of the algorithm.

### Sample Code
```c++
// AbstractClass
class Speaker {
public:
	virtual ~Speaker() = default;

	virtual void speak() {
		openMyMouth();
		doSpeakMyLanguage();
		closeMyMouth();
	}
protected:
	Speaker() = default;

	virtual void doSpeakMyLanguage() = 0;
private:
	void openMyMouth() {
		std::cout << "The speaker starts to speak: ";
	}
	void closeMyMouth() {
		std::cout << "\nThe speaker just finished speaking\n" << std::endl;
	}
};


// ConcreteClass
class EnglishSpeaker : public Speaker {
protected:
	void doSpeakMyLanguage() override {
		std::cout << "Hi, it's a good day!";
	}
};

class KoreanSpeaker : public Speaker {
protected:
	void doSpeakMyLanguage() override {
		std::cout << "안녕하세요, 좋은 날입니다!";
	}
};




// some codes
Speaker* speaker = nullptr;

speaker = new EnglishSpeaker();
speaker->speak();
delete speaker;

speaker = new KoreanSpeaker();
speaker->speak();
delete speaker;

/*
print result
The speaker starts to speak: Hi, it's a good day!
The speaker just finished speaking

The speaker starts to speak: 안녕하세요, 좋은 날입니다!
The speaker just finished speaking
*/

```

### Implementation
Three implementation issues are worth noting:
1. *Using C++ access control*
    * In C++, the primitive operations that a template method calls can be declared protected members.
        + This ensures that they are only called by the template method.
    * Primitive operations that must be overridden are declared pure virtual.
        + The template method itself should not be overridden;
            - therefore you can make the template method a nonvirtual member function.
2. *Minimizing primitive operations*
    * An important goal in designing template methods is to minimize the number of primitive operations that a subclass must override to flesh out the algorithm. 
    * The more operations that need overriding, the more tedious things get for clients.
3. *Naming conventions*
    * You can identify the operations that should be overridden by adding a prefix to their names.
    * For example, the MacApp framework for Macintosh applications prefixes template method names with `"Do-"`
        + `DoCreateDocument()`, `DoRead()`, and so forth

### Related Patterns
- **Factory Method** are often called by template methods.
- **Strategy**: Template methods use inheritance to vary part of an algorithm.
    * Strategies use delegation to vary the entire algorithm.


## Consequences
Template methods are a fundamental technique for code reuse.
- They are particularly important in class libraries, because they are the means for factoring out common behavior in library classes.
- Template methods call the following kinds of operations:
    * concrete operations (either on the ConcreteClass or on client classes);
    * concrete AbstractClass operations (i.e., operations that are generally useful to subclasses);
    * primitive operations (i.e., abstract operations);
    * factory methods (see Factory Method); and
    * **hook operations**, which provide default behavior that subclasses can extend if necessary.
        + A hook operation often does nothing by default.
- It's important for template methods to specify which operations are hooks (*may* be overridden) and which are abstract operations (*must* be overridden).
    * To reuse an abstract class effectively, subclass writers must understand which operations are designed for overriding.
- A subclass can *extend* a parent class operation's behavior by overriding the operation and calling the parent operation explicitly:
    ```c++
    void DerivedClass::operation() {
        // DerivedClass extended behavior
        ParentClass::operation();
    }
    ```
- Unfortunately, it's easy to forget to call the inherited operation.
    * We can transform such an operation into a template method to give the parent control over how subclasses extend it.
    * The idea is to call a hook operation from a template method in the parent class.
    * Then subclasses can then override this hook operation:
        ```c++
        void ParentClass::operation() {
            // ParentClass behavior
            hookOperation();
        }
        ```
- `hookOperation()` does nothing in ParentClass:
    ```c++
    void ParentClass::hookOperation() { }
    ```
- Subclasses override `hookOperation` to extend its behavior:
    ```c++
    void DerivedClass::hookOperation() {
        // derived class extension
    }
    ```

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}