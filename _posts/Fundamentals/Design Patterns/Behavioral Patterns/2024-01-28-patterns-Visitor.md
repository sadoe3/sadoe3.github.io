---
title: "Design Patterns : Visitor"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, Visitor]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-28
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Visitor


## Problem

### Intent
Represent an operation to be performed on the elements of an object structure. Visitor lets you define a **new operation without changing the classes of the elements** on which it operates.

### Applicability
Use the **Visitor** pattern when
- **an object** structure contains many classes of **objects** with **differing interfaces**, and you want to perform **operations on these objects** that depend on their concrete classes.
- many distinct and unrelated operations need to be performed on objects in an object structure, and you want to avoid "polluting" their classes with these operations.
    * Visitor lets you keep related operations together by defining them in one class. 
    * When the object structure is shared by many applications, use Visitor to put operations in just those applications that need them.
- the classes defining the object structure rarely change, but you often want to define new operations over the structure.
    * Changing the object structure classes requires redefining the interface to all visitors, which is potentially costly.
    * If the object structure classes change often, then it's probably better to define the operations in those classes.


## Solution

### Motivation
Suppose that you want to implement the code covered from the [**Composite**](https://sadoe3.github.io/design-patterns/patterns-Composite/) pattern in a different way.
- The **problem** is that, in this case, you want to add a new operation for the concrete items without changing the interface of them.
    * the operations usually **traverse** all the items in player's inventory
- A **solution** is to define 2 class hierarchies:
    * one for the operations
        + the base class of the operation classes must have the method named `visit(ConcreteItem*)` which does its job for the given item.
    * the other for the items
        + the base class of the item classes must have the method named `accept(Visitor&)` which calls `visitor.visit(this)` if the caller is the leaf node, otherwise it calls `accept()` for its children.


### Participants
- **Visitor** (`ItemVisitor`)
    * declares a `visit()` operation for each class of Concrete Element in the object structure.
        + The operation's name and signature identifies the class that sends the `visit()` request to the visitor.
        + That lets the visitor determine the concrete class of the element being visited.
        + Then the visitor can access the element directly through its particular interface.
- **Concrete Visitor** (`ItemVisitorPrice`, `ItemVisitorName`)
    * implements each operation declared by Visitor.
        + Each operation implements a fragment of the algorithm defined for the corresponding class of object in the structure.
        + Concrete Visitor provides the context for the algorithm and stores its local state.
        + This state often accumulates results during the traversal of the structure.
- **Element** (`Item`)
    * defines an `accept()` operation that takes a visitor as an argument.
- **ConcreteElement** (`Inventory`, `Consumable`, `Sword`, `Bow`, etc.)
    * implements an `accept()` operation that takes a visitor as an argument.
- **ObjectStructure** (`playerInventory`)
    * can enumerate its elements.
    * may provide a high-level interface to allow the visitor to visit its elements.
    * may either be a composite (see **Composite**) or a collection such as a list or a set.

### Sample Code
```c++
#include <iostream>
#include <map>
#include <forward_list>

// this code exists for test purpose only
// this is not related to Visitor pattern
struct ItemData {
	std::string name;
	unsigned price;
};

enum class ItemID { Inventory, Consumable, Equipment, Defense, Attack, Potion, Helmet, Armor, Sword, Bow};

std::map<unsigned, ItemData> dataBase;
void initDB() {
	dataBase[static_cast<int>(ItemID::Inventory)] = ItemData{"Inventory", 0};
	dataBase[static_cast<int>(ItemID::Consumable)] = ItemData{ "Consumable", 0 };
	dataBase[static_cast<int>(ItemID::Equipment)] = ItemData{ "Equipment", 0 };
	dataBase[static_cast<int>(ItemID::Defense)] = ItemData{ "Defense", 0 };
	dataBase[static_cast<int>(ItemID::Attack)] = ItemData{ "Attack", 0 };

	dataBase[static_cast<int>(ItemID::Potion)] = ItemData{ "Potion", 10 };
	dataBase[static_cast<int>(ItemID::Helmet)] = ItemData{ "Helmet", 32 };
	dataBase[static_cast<int>(ItemID::Armor)] = ItemData{ "Armor", 200 };
	dataBase[static_cast<int>(ItemID::Sword)] = ItemData{ "Sword", 1 };
	dataBase[static_cast<int>(ItemID::Bow)] = ItemData{ "Bow", 5000 };
}




class ItemVisitor;
// Component
// Element
class Item {
public:
	Item(const unsigned& inputID) : id(inputID) {}
	virtual ~Item() { std::cout << getItemName() << " is deleted" << std::endl; }

	virtual void accept(ItemVisitor&) = 0;		// necessary code for Visitor

	// name, min level, price are returned by using database
	std::string getItemName() {
		return dataBase[id].name;
	}
	unsigned getPrice() {
		return dataBase[id].price;
	}

	virtual void addItem(Item*) { /* empty definition by default */ }
	virtual void removeItem(Item*) { /* empty definition by default */ }
protected:
	unsigned id;
};



// Composite
class ItemComposite : public Item {
public:
	ItemComposite(const unsigned& inputID) : Item(inputID) {}
	~ItemComposite() {
		for (auto curItem : items)
			delete curItem;
	}

	void addItem(Item* newItem) override {
		items.push_front(newItem);
	}
	void removeItem(Item* targetItem) override {
		items.remove(targetItem);
	}
protected:
	std::forward_list<Item*> items;
};




class Inventory;
class Consumable;
class Equipment;
class Defense;
class Attack;
class Potion;
class Helmet;
class Armor;
class Sword;
class Bow;
// Visitor
// necessary code for Visitor
class ItemVisitor {
public:
	virtual ~ItemVisitor() = default;

	// polymorphism!
	virtual void visit(Item*) = 0;
protected:
	ItemVisitor() = default;
};

// ConcreteElement
// Composite
class Inventory : public ItemComposite {        // component
public:
	Inventory() : ItemComposite(static_cast<int>(ItemID::Inventory)) {}

	// necessary code for Visitor
	void accept(ItemVisitor& visitor) override {
		// for statement is needed for composite classes
		for (auto currentItem : items)
			currentItem->accept(visitor);

		visitor.visit(this);
	}
};
class Consumable : public ItemComposite {
public:
	Consumable() : ItemComposite(static_cast<int>(ItemID::Consumable)) {}

	// necessary code for Visitor
	void accept(ItemVisitor& visitor) override {
		// for statement is needed for composite classes
		for (auto currentItem : items) {
			currentItem->accept(visitor);
		}

		visitor.visit(this);
	}
};
class Equipment : public ItemComposite {
public:
	Equipment() : ItemComposite(static_cast<int>(ItemID::Equipment)) {}

	// necessary code for Visitor
	void accept(ItemVisitor& visitor) override {
		// for statement is needed for composite classes
		for (auto currentItem : items)
			currentItem->accept(visitor);

		visitor.visit(this);
	}
};
class Defense : public ItemComposite {
public:
	Defense() : ItemComposite(static_cast<int>(ItemID::Defense)) {}

	// necessary code for Visitor
	void accept(ItemVisitor& visitor) override {
		// for statement is needed for composite classes
		for (auto currentItem : items)
			currentItem->accept(visitor);

		visitor.visit(this);
	}
};
class Attack : public ItemComposite {
public:
	Attack() : ItemComposite(static_cast<int>(ItemID::Attack)) {}

	// necessary code for Visitor
	void accept(ItemVisitor& visitor) override {
		// for statement is needed for composite classes
		for (auto currentItem : items)
			currentItem->accept(visitor);

		visitor.visit(this);
	}
};

// Leaf
class Potion : public Item {
public:
	Potion() : Item(static_cast<int>(ItemID::Potion)) {}

	// necessary code for Visitor
	void accept(ItemVisitor& visitor) override {
		visitor.visit(this);
	}
};
class Helmet : public Item {
public:
	Helmet() : Item(static_cast<int>(ItemID::Helmet)) {}

	// necessary code for Visitor
	void accept(ItemVisitor& visitor) override {
		visitor.visit(this);
	}
};
class Armor : public Item {
public:
	Armor() : Item(static_cast<int>(ItemID::Armor)) {}

	// necessary code for Visitor
	void accept(ItemVisitor& visitor) override {
		visitor.visit(this);
	}
};
class Sword : public Item {
public:
	Sword() : Item(static_cast<int>(ItemID::Sword)) {}

	// necessary code for Visitor
	void accept(ItemVisitor& visitor) override {
		visitor.visit(this);
	}
};
class Bow : public Item {
public:
	Bow() : Item(static_cast<int>(ItemID::Bow)) {}

	// necessary code for Visitor
	void accept(ItemVisitor& visitor) override {
		visitor.visit(this);
	}
};


// ConcreteVisitor
// necessary code for Visitor
class ItemVisitorPrice : public ItemVisitor {
public:
	unsigned getTotalPrice() {
		auto returnedValue = total;
		// reset total for reuse
		total = 0;
		return returnedValue;
	}

	void visit(Item* target) override {
		total += target->getPrice();
	}

private:
	unsigned total = 0;
};
class ItemVisitorName : public ItemVisitor {
public:
	void visit(Item* target) override {
		std::cout << target->getItemName() << " ";
	}
};





// some codes
initDB();

Inventory* playerInventory = new Inventory;
Consumable* consumable = new Consumable;
Equipment* equipment = new Equipment;
playerInventory->addItem(consumable);
playerInventory->addItem(equipment);

Defense* defense = new Defense;
Attack* attack = new Attack;
equipment->addItem(defense);
equipment->addItem(attack);

Potion* potion = new Potion;
consumable->addItem(potion);

Helmet* helmet = new Helmet;
Armor* armor = new Armor;
defense->addItem(helmet);
defense->addItem(armor);

Sword* sword = new Sword;
Bow* bow = new Bow;
attack->addItem(sword);
attack->addItem(bow);



// necessary code for Visitor
// get total price
ItemVisitorPrice priceVisitor;
playerInventory->accept(priceVisitor);
std::cout << "total price: " << priceVisitor.getTotalPrice() << std::endl;

// get total name
ItemVisitorName nameVisitor;
playerInventory->accept(nameVisitor);

std::cout << "\n" << std::endl;
delete playerInventory;


/*
print result
total price: 5243
Bow Sword Attack Armor Helmet Defense Equipment Potion Consumable Inventory

Bow is deleted
Sword is deleted
Attack is deleted
Armor is deleted
Helmet is deleted
Defense is deleted
Equipment is deleted
Potion is deleted
Consumable is deleted
Inventory is deleted
*/
```

### Implementation
- Each object structure will have an associated Visitor class.
    * This abstract visitor class declares a `visitConcreteElement()` operation for each class of ConcreteElement defining the object structure.
    * Each `visit()` operation on the Visitor declares its argument to be a particular ConcreteElement, allowing the Visitor to access the interface of the ConcreteElement directly.
    * Concrete Visitor classes override each `visit()` operation to implement visitor-specific behavior for the corresponding ConcreteElement class.
- The Visitor class would be declared like this in C++:
    ```c++
    class Visitor {
    public:
        virtual void visitElementA(ElementA*);
        virtual void visitElementB(ElementB*);

        // and so on for other concrete elements
    protected:
        Visitor();
    };
    ```
- Each class of ConcreteElement implements an `accept()` operation that calls the matching `visit`... operation on the visitor for that ConcreteElement.
    * Thus the operation that ends up getting called depends on both the class of the element and the class of the visitor.
- The concrete elements are delcared as
    ```c++
    class Element {
    public:
        virtual ~Element();
        virtual void accept(Visistor&) = 0;
    protected:
        Element();
    };

    class ElementA : public Element {
    public:
        void accept(Visistor &v) override { v.visitElementA(this); } 
    };  

    class ElementB : public Element {
    public:
        void accept(Visistor &v) override { v.visitElementB(this); } 
    };  
    ```
- A CompositeElement class might implement `accept()` like this:
    ```c++
    class CompositeElement : public Element {
    public:
        void accept(Visitor&) override;
    private:
    	std::forward_list<Element*> elements;
    };

    void CompositeElement::accept(Visitor &v) {
		for (auto currentElement : elements)
			currentElement->accept(visitor);

        v.visitCompositeElement(this);
    }
    ```
- Here are two other implementation issues that arise when you apply the Visitor pattern:
1. *Double dispatch*
    * Effectively, the Visitor pattern lets you add operations to classes without changing them.
        +  Visitors achieves this by using a technique called **double-dispatch**.   
    * Languages like C++ and smalltalk support **single-dispatch**.
        +  In single-dispatch languages, two criteria determine which operation will fulfill a request:
            - the name of the request and
            - the type of receiver.
        + The operation that gets executed depends both on the kind of request and the type of the receiver.
    * "Double-dispatch" simply means the operation that gets executed depends on the kind of request and the types of **two receivers**.
        + `accept()` is a double-dispatch operation.
            - Its meaning depends on two types: the Visitor's and the Element's. 
        + Double-dispatching lets visitors request different operations on each class of element."
    * This is the key to the Visitor pattern:
        + The operation that gets executed depends on both the type of Visitor and the type of Element it visits.
    * Instead of binding operations statically into the Element interface, you can consolidate the operations in a Visitor and use `accept()` to do the binding at run-time.
        + Extending the Element interface amounts to defining one new Visitor subclass rather than many new Element subclasses.
2. *Who is responsible for traversing the object structure?*
    * A visitor must visit each element of the object structure.
        + The question is, how does it get there?
    * We can put responsibility for traversal in any of three places:
        + in the object structure,
        + in the visitor,
        + or in a separate iterator object (see **Iterator**).
    * Often the object structure is responsible for iteration.
        + A collection will simply iterate over its elements, calling the `accept()` operation on each.
        + A composite will commonly traverse itself by having each `accept()` operation traverse the element's children and call `accept()` on each of them recursively.
    * Another solution is to use an iterator to visit the elements.
        + In C++, you could use either an internal or external iterator, depending on what is available and what is most efficient.
        + The main difference is that an internal iterator will not cause double-dispatching;
            - it will call an operation on the visitor with an element as an argument as opposed to calling an operation on the element with the visitor as an argument.
        + But it's easy to use the Visitor pattern with an internal iterator if the operation on the visitor simply calls the operation on the element without recursing.
    * You could even put the traversal algorithm in the visitor, although you'll end up duplicating the traversal code in each Concrete Visitor for each aggregate Concrete Element.
        + The main reason to put the traversal strategy in the visitor is to implement a particularly complex traversal, one that depends on the results of the operations on the object structure.


### Related Patterns
- **Composite**: Visitors can be used to apply an operation over an object structure defined by the Composite pattern.
- **Interpreter**: Visitor may be applied to do the interpretation.


## Consequences
Some of the benefits and liabilities of the Visitor pattern are as follows:
1. *Visitor makes adding new operations easy*
    * You can define a **new operation** over an object structure **simply** by adding a **new visitor**.
    * In contrast, if you spread functionality over many classes, then you must change each class to define a new operation.
2. *Adding new Concrete Element classes is hard*
    * The Visitor pattern makes it **hard to add new subclasses of Element**.
        + Each **new ConcreteElement** gives rise to a **new abstract operation on Visitor** and a corresponding implementation in **every Concrete Visitor** class.
    * Sometimes a default implementation can be provided in Visitor that can be inherited by most of the Concrete Visitors
        + but this is the exception rather than the rule.
    * So the key consideration in applying the Visitor pattern is whether you are mostly likely to change
        + the algorithm applied over an object structure
        + or the classes of objects that make up the structure.
    * The Visitor class hierarchy can be difficult to maintain when new ConcreteElement classes are added frequently.
        + In such cases, it's probably easier just to define operations on the classes that make up the structure.
    * If the Element class hierarchy is stable, but you are continually adding operations or changing algorithms,
        + then the Visitor pattern will help you manage the changes.
3. *Visiting across class hierarchies*
    * An iterator (see Iterator) can visit the objects in a structure as it traverses them by calling their operations.
        + But an iterator can't work across object structures with different types of elements.
    * For example, the following Iterator interface can access only objects of type `Item`:
        ```c++
        template <class Item>
        class Iterator {
            // ...
            Item getCurrentItem() const;
        };
        ```
        + This implies that all elements the iterator can visit have a common parent class Item.
    * Visitor does not have this restriction.
        + It **can visit objects that don't have a common parent class**.
        + You can add any type of object to a Visitor interface.
    * For example, 
        ```c++
        class Visitor {
        public:
            // ...
            void visitMyType(MyType*);
            void visitYourType(YourType*);
        };
        ```
    * `MyType` and `YourType` do not have to be related through inheritance at all.
4. *Accumulating state*
    * Visitors can accumulate state as they visit each element in the object structure.
    * Without a visitor, this state would be passed as extra arguments to the operations that perform the traversal, or they might appear as global variables.
        + and this doesn't seem efficient.
5. *Breaking encapsulation*
    * Visitor's approach assumes that the ConcreteElement interface is powerful enough to let visitors do their job.
        + As a result, the pattern often forces you to provide public operations that access an element's internal state, which may compromise its encapsulation.
    * You might solve this by making friendship between them.


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}