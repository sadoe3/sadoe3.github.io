---
title: "Design Patterns : Factory Method"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Creational Pattern, Factory Method]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-18
---

# Creational Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Factory Method

### Also Known As
Virtual Constructor


## Problem

### Intent
Define an interface for creating an object, but let **subclasses decide which class to instantiate**. Factory Method lets a class defer instantiation to subclasses.

### Applicability
Use the **Factory Method** pattern when
- a class can't anticipate the class of objects it must create.
- a class wants its **subclasses to specify** the objects it creates.
- classes **delegate** responsibility to one of several helper subclasses, and you want to localize the knowledge of which helper subclass is the delegate.


## Solution

### Motivation
Suppose that you want to create a manual for a certain product in multiple languages.
- The **problem** is that the `ManualCreator` class which handles the creation of the manual **knows what kind of information** which the manual should contain, but it does **not know how to express it in a specific language**   
- A **solution** is to **delgate** the responsibility of the language to the derived classes from the `ManualCreator` class.
    * The derived classes do **not override** the operation which returns the `Manual` object
    * but they **do override** the **factory methods** which add the certain information to the `Manual` object

### Participants
- **Creator** (`ManualCreator`)
    * declares the factory method, which returns an object of type Product.
        + Creator may also define a default implementation of the factory method that returns a default ConcreteProduct object.
    * may call the factory method to create a Product object.
- **ConcreteCreator** (`EnglishManualCreator`, `KoreanManualCreator`)
    * overrides the factory method to return an instance of a ConcreteProduct
- **Product** (`Manual`)
    * defines the interface of objects the factory method creates.

### Sample Code
```c++
struct Manual {
	std::string overview;
	std::string caution;
	std::string howTo;
};

class ManualCreator {
public:
	Manual* createManual() {
		Manual* newManual = new Manual();
		addOverview(*newManual);
		addCaution(*newManual);
		addHowTo(*newManual);

		return newManual;
	}
protected:
	virtual void addOverview(Manual& manual) = 0;
	virtual void addCaution(Manual& manual) = 0;
	virtual void addHowTo(Manual& manual) = 0;
};

class EnglishManualCreator : public ManualCreator {
protected:
	void addOverview(Manual& manual) override {
		manual.overview = "This is English overview";
	}
	void addCaution(Manual& manual) override {
		manual.caution = "This is English caution";
	}
	void addHowTo(Manual& manual) override {
		manual.howTo = "This is English how to";
	}
};

class KoreanManualCreator : public ManualCreator {
protected:
	void addOverview(Manual& manual) override {
		manual.overview = "이것은 한국어 개요입니다";
	}
	void addCaution(Manual& manual) override {
		manual.caution = "이것은 한국어 주의사항입니다";
	}
	void addHowTo(Manual& manual) override {
		manual.howTo = "이것은 한국어 사용법입니다";
	}
};



// some codes
ManualCreator* creator = nullptr;

switch (/*condition for current location*/) {
case Locale::EN:
    creator = new EnglishManualCreator();
    break;
case Locale::KR:
    creator = new KoreanManualCreator();
    break;
}

if (creator) {
    Manual* manual = creator->createManual();
    std::cout << manual->howTo << std::endl;
}

delete creator;
```

### Implementation
Consider the following issues when applying the Factory Method pattern:
1. *Two major varieties*
    * The two main **variations** of the Factory Method pattern are
        + (1) the case when the `Creator` class is an **abstract class** and does **not** provide an **implementation** for the factory method it declares, and
        + (2) the case when the `Creator` is a **concrete class** and provides a **default implementation** for the factory method.
2. *Parameterized factory methods*
    * Another **variation** on the pattern lets the factory method create **multiple** kinds of **products**.
        + The factory method takes a parameter that identifies the kind of object to create.
        + All objects the factory method creates will share the Product interface.
    * On the other hand, you can make the operation which returns the product as `virtual` so that the derived classes can return the corresponding **ConcreteProduct**s. 
3. *Language-specific variants and issues*
    * Factory methods in C++ are always virtual functions and are often pure virtual.
    * Just be careful not to call factory methods in the `Creator`'s constructor-the factory method in the `ConcreteCreator` won't be available yet.
4. *Using templates to avoid subclassing*
    * As we've mentioned, another potential problem with factory methods is that they might force you to subclass just to create the appropriate Product objects. 
    * Another way to get around this in C++ is to provide a **template** subclass of `Creator` that's parameterized by the Product class:
        ```c++
        class Creator { 
        public:
            virtual Product* createProduct() = 0;
        };


        template ‹class TheProduct>
        class StandardCreator: public Creator {
        public:
            virtual Product* createProduct();
        };

        template <class TheProduct>
        Product* StandardCreator<TheProduct>::createProduct() {
            return new TheProduct;
        }


        class MyProduct: public Product {
        public:
            MyProduct();
            //...
        };
        StandardCreator<MyProduct> myCreator;
        ```
5. *Naming conventions*
    * It's good practice to use **naming conventions** that make it **clear** you're using factory methods.

### Related Patterns
- **Abstract Factory** is often implemented with factory methods.
- Factory methods are usually called within **Template** Methods.
- **Prototype**s don't require subclassing `Creator`.
    * However, they often require an Initialize operation on the Product class. `Creator` uses Initialize to initialize the object.
    * Factory Method doesn't require such an operation.


## Consequences
Factory methods eliminate the need to bind application-specific classes into your code.
- The code **only** deals with the **Product interface**;
    * therefore it can work with **any user-defined ConcreteProduct classes**.
- A potential **disadvantage** of factory methods is that
    * clients might have to subclass the `Creator` class just to create a particular ConcreteProduct object.
    * Subclassing is fine when the client has to subclass the `Creator` class anyway
        + but otherwise the client now must deal with another point of evolution.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}