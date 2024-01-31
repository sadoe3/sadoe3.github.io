---
title: "Design Patterns : Observer"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, Observer]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-26
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Observer

### Also Known As
Dependents, Publish-Subscribe


## Problem

### Intent
Define a **one-to-many** dependency between objects so that when **one** object **changes** state, **all** its dependents are **notified and updated automatically**

### Applicability
Use the **Observer** pattern in any of the following situations:
- When an abstraction has two aspects, one dependent on the other.
    * Encapsulating these aspects in separate objects lets you vary and reuse them independently.
- When a change to **one** object requires **changing others**, and you **don't know how many** objects need to be changed.
- When an object should be able to notify other objects without making assumptions about who these objects are.
    * In other words, you do **not want** these objects **tightly coupled**.


## Solution

### Motivation
Suppose that you want to implement a simple program which prints a current shape in two languages : English and Korean.
- The **problem** is that you want to make two speakers speak right after the current shape gets changed.
- A **solution** is to make one-to-many dependency through inheritance and composition.

### Participants
- **Subject** (`Subject`)
    * knows its observers. Any number of Observer objects may observe a subject.
    * provides an interface for attaching and detaching Observer objects.
- **Observer** (`Observer`)
    * defines an updating interface for objects that should be notified of changes in a subject.
- **ConcreteSubject** (`ShapeHolder`)
    * stores state of interest to ConcreteObserver objects.
    * sends a notification to its observers when its state changes.
- **ConcreteObserver** (`EnglishSpeaker`, `KoreanSpeaker`)
    * maintains a reference to a ConcreteSubject object.
    * stores state that should stay consistent with the subject's.
    * implements the Observer updating interface to keep its state consistent with the subject's.

### Sample Code
```c++
// this code exists for test purpose only
// this is not related to Observer pattern
enum class ShapeType { Circle, Rectangle, Triangle, LastShapeType };


class Subject;
// Observer
class Observer {
public:
	virtual ~Observer() = default;
	virtual void update(Subject* targetSubject) = 0;
protected:
	Observer() = default;
};

// Subject
class Subject {
public:
	virtual ~Subject() = default;

	void attach(Observer* observer) {
		observers.insert(observer);
	}
	void detach(Observer* observer) {
		observers.erase(observer);
	}
	void notify() {
		for (auto currentObserver : observers)
			currentObserver->update(this);
	}
protected:
	Subject() = default;
private:
	std::set<Observer*> observers;
};


// ConcreteSubject
class ShapeHolder : public Subject {
public:
	ShapeType getShape() {
		return shape;
	}
	void setShape(const ShapeType & inputShape) {
		shape = inputShape;
		notify();
	}
private:
	ShapeType shape = ShapeType::LastShapeType;
};


// ConcreteObserver
class Speaker {
public:
	virtual ~Speaker() = default;
	virtual void speak() = 0;
};

class EnglishSpeaker : public Speaker, public Observer {
public:
	EnglishSpeaker(ShapeHolder* inputSubject) : subject(inputSubject) {
		subject->attach(this);
	}
	~EnglishSpeaker() {
		subject->detach(this);
	}

	void update(Subject* targetSubject) override {
		if (targetSubject == subject)
			speak();
	}
	void speak() override;
private:
	ShapeHolder* subject;
};
void EnglishSpeaker::speak() {
	std::string result = "Wrong Shape";
	switch (subject->getShape()) {
	case ShapeType::Circle:
		result = "Circle";
		break;
	case ShapeType::Rectangle:
		result = "Rectangle";
		break;
	case ShapeType::Triangle:
		result = "Triangle";
		break;
	}
	std::cout << "English : " << result << std::endl;;
}

class KoreanSpeaker : public Speaker, public Observer {
public:
	KoreanSpeaker(ShapeHolder* inputSubject) : subject(inputSubject) {
		subject->attach(this);
	}
	~KoreanSpeaker() {
		subject->detach(this);
	}

	void update(Subject* targetSubject) override {
		if (targetSubject == subject)
			speak();
	}
	void speak() override;
private:
	ShapeHolder* subject;
};
void KoreanSpeaker::speak() {
	std::string result = "잘못된 모양";
	switch (subject->getShape()) {
	case ShapeType::Circle:
		result = "동그라미";
		break;
	case ShapeType::Rectangle:
		result = "네모";
		break;
	case ShapeType::Triangle:
		result = "세모";
		break;
	}
	std::cout << "Korean : " << result << std::endl;;
}





// some codes
ShapeHolder* shape = new ShapeHolder();

std::set<Speaker*> speakers;
speakers.insert(new EnglishSpeaker(shape));
speakers.insert(new KoreanSpeaker(shape));


shape->setShape(ShapeType::Circle);
std::cout << "\n" << std::endl;
shape->setShape(ShapeType::Rectangle);
std::cout << "\n" << std::endl;
shape->setShape(ShapeType::LastShapeType);
std::cout << "\n" << std::endl;
shape->setShape(ShapeType::Triangle);


for (auto currentSpeaker : speakers)
    delete currentSpeaker;
delete shape;
/*
print result
English : Circle
Korean : 동그라미


English : Rectangle
Korean : 네모


English : Wrong Shape
Korean : 잘못된 모양


English : Triangle
Korean : 세모
*/
```

### Implementation
Several issues related to the implementation of the dependency mechanism are discussed in this section.
1. *Mapping subjects to their observers*
    * The simplest way for a subject to keep track of the observers it should notify is to store references to them explicitly in the subject.
    * However, such storage may be too expensive when there are many subjects and few observers.
        + One solution is to trade space for time by using an associative look-up (e.g., a hash table) to maintain the subject-to-observer mapping.
        + Thus a subject with no observers does not incur storage overhead.
        + On the other hand, this approach increases the cost of accessing the observers.
2. *Observing more than one subject*
    * It might make sense in some situations for an observer to depend on more than one subject.
    * It's necessary to extend the `update()` interface in such cases to let the observer know which subject is sending the notification.
        + The subject can simply pass itself as a parameter in the `update()` operation, thereby letting the observer know which subject to examine.
3. *Who triggers the update?*
    * The subject and its observers rely on the notification mechanism to stay consistent.
    * But what object actually calls `notify()` to trigger the update? Here are two options:
        + (a) Have state-setting operations on Subject call `notify()` after they change the subject's state.
            - The advantage of this approach is that clients don't have to remember to call `notify()` on the subject.
            - The disadvantage is that several consecutive operations will cause several consecutive updates, which may be inefficient.
        + (b) Make clients responsible for calling `notify()` at the right time
            -  The advantage here is that the client can wait to trigger the update until after a series of state changes has been made, thereby avoiding needless intermediate updates.
            - The disadvantage is that clients have an added responsibility to trigger the update. That makes errors more likely, since clients might forget to call `notify()`.
4. *Dangling references to deleted subjects*
    * Deleting a subject should not produce dangling references in its observers.
    * One way to avoid dangling references is to make the subject notify its observers as it is deleted so that they can reset their reference to it.
    * In general, simply deleting the observers is not an option, because other objects may reference them, or they may be observing other subjects as well.
5. *Making sure Subject state is self-consistent before notification*
    * It's important to make sure Subject state is self-consistent before calling `notify()`, because observers query the subject for its current state in the course of updating their own state.
        + This self-consistency rule is easy to violate unintentionally when Subject subclass operations call inherited operations.
    * For example, the notification in the following code sequence is trigged when the subject is in an inconsistent state:
        ```c++
        void MySubject::operation(int newValue) {
            BaseClassSubject::operation(newValue);
            // trigger notification

            myInstVar += newValue;
            // update subclass state (too late!)
        }
        ```
    * You can avoid this pitfall by sending notifications from template methods (**Template Method**) in abstract Subject classes.
        + Define a primitive operation for subclasses to override, and make `notify()` the last operation in the template method, which will ensure that the object is self-consistent when subclasses override Subject operations.
    * By the way, it's always a good idea to document which Subject operations trigger notifications.
6. *Avoiding observer-specific update protocols: the push and pull models*
    * Implementations of the Observer pattern often have the subject broadcast additional information about the change.
        + The subject passes this information as an argument to `update()`.
        + The amount of information may vary widely.
    * At one extreme, which we call the **push model**,
        + the subject sends observers detailed information about the change, whether they want it or not.
        + The push model might make observers less reusable, because Subject classes make assumptions about Observer classes that might not always be true.
    * At the other extreme is the **pull model**;
        + the subject sends nothing but the most minimal notifcation, and observers ask for details explicitly thereafter.
        + The pull model may be inefficient, because Observer classes must ascertain what changed without help from the Subject.
7. *Specifying modifications of interest explicitly*
    * You can improve update efficiency by extending the subject's registration interface to allow registering observers only for specific events of interest. 
        + When such an event occurs, the subject informs only those observers that have registered interest in that event.
        + One way to support this uses the notion of **aspects** for Subject objects.
    * To register interest in particular events, observers are attached to their subjects using
        ```c++
        void Subject::attach(Observer*, Aspect& interest);    
        ```
    * where interest specifies the event of interest.
        + At notification time, the subject supplies the changed aspect to its observers as a parameter to the Update operation.
    * For example:
        ```c++
        void Observer::update(Subject*, Aspect& interest);
        ```
8. *Encapsulating complex update semantics*
    * When the dependency relationship between subjects and observers is particularly complex, an object that maintains these relationships might be required.
        + We call such an object a **ChangeManager**.
        + Its purpose is to minimize the work required to make observers reflect a change in their subject.
    * For example, if an operation involves changes to several interdependent subjects, you might have to ensure that their observers are notified only after all the subjects have been modified to avoid notifying observers more than once.
    * ChangeManager has three responsibilities:
        + (a) It maps a subject to its observers and provides an interface to maintain this mapping.
            - This eliminates the need for subjects to maintain references to their observers and vice versa.
        + (b) It defines a particular update strategy.
        + (c) It updates all dependent observers at the request of a subject.
    * There are two specialized ChangeManagers.
        + SimpleChangeManager is naive in that it always updates all observers of each subject.
            - SimpleChangeManager is fine when multiple updates aren't an issue.
        + In contrast, DAGChangeManager handles directed-acyclic graphs of dependencies between subjects and their observers.
            - A DAGChangeManager is preferable to a SimpleChangeManager when an observer observes more than one subject.
            - In that case, a change in two or more subjects might cause redundant updates.
            - The DAGChangeManager ensures the observer receives just one update. 
    * ChangeManager is an instance of the **Mediator** pattern.
    * In general there is only one ChangeManager, and it is known globally.
        The **Singleton** pattern would be useful here.

### Related Patterns
- **Mediator**: By encapsulating complex update semantics, the ChangeManager acts as mediator between subjects and observers.
- **Singleton**: The ChangeManager may use the Singleton pattern to make it unique and globally accessible.


## Consequences
The Observer pattern lets you vary subjects and observers independently.
- You can reuse subjects without reusing their observers, and vice versa.
    * It lets you add observers without modifying the subject or other observers.
- Further benefits and liabilities of the Observer pattern include the following:
1. *Abstract coupling between Subject and Observer*
    * All a subject knows is that it has a list of observers, each conforming to the simple interface of the abstract Observer class.
        + The subject doesn't know the concrete class of any observer.
        + Thus the coupling between subjects and observers is abstract and minimal.
2. *Support for broadcast communication*
    * Unlike an ordinary request, the notification that a subject sends needn't specify its receiver.
        + The notification is broadcast automatically to all interested objects that subscribed to it.
    * This gives you the freedom to add and remove observers at any time.
        + It's up to the observer to handle or ignore a notification.
3. *Unexpected updates*
    * Because observers have no knowledge of each other's presence, they can be blind to the ultimate cost of changing the subject.
        + A seemingly innocuous operation on the subject may cause a cascade of updates to observers and their dependent objects.
        + Moreover, dependency criteria that aren't well-defined or maintained usually lead to spurious updates, which can be hard to track down.
    * This problem is aggravated by the fact that the simple update protocol provides no details on what changed in the subject.
        + Without additional protocol to help observers discover what changed, they may be forced to work hard to deduce the changes.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}