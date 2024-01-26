---
title: "Design Patterns : Chain of Responsibility"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, Chain of Responsibility]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-23
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Chain of Responsibility


## Problem

### Intent
**Avoid coupling** the **sender** of a **request** to its **receiver** by giving more than one object a chance to handle the request. **Chain** the receiving objects and pass the request along the chain until an object handles it.

### Applicability
Use **Chain of Responsibility** when
- more than one object may handle a request, and the handler isn't known a *priori*.
    * The handler should be ascertained automatically.
- you want to issue a **request** to one of **several** objects **without specifying** the receiver explicitly.
- the set of objects that can handle a request should be specified **dynamically**.


## Solution

### Motivation
Suppose that you want to implement a defense system against the canon. There are 3 places where the shields can be installed.
- The **problem** is that the **proper** shield **object** which handles the canon is **not known** when the canon is fired
- A **solution** is to implement shield classes as a **chain**
    * so that the fired ball gets passed along a **chain of objects** until one of them handles (defenses) it

### Participants
- **Handler** (`ShieldHandler`)
    * defines an interface for handling requests.
    * (optional) implements the successor link.
- **ConcreteHandler** (`InnerShield`, `MiddleShield`, `OuterShield`)
    * handles requests it is responsible for.
    * can access its successor.
    * if the ConcreteHandler can handle the request, it does so; otherwise it forwards the request to its successor.
- **Client** 
    * initiates the request to a ConcreteHandler object on the chain.

### Sample Code
```c++
// this code exists for test purpose only
// this is not related to Chain of Responsibility pattern
class ShieldHandler;
enum class ShieldType { Inner, Middle, Outer, LastShieldType };

class ShieldManager {
public:
	static void defense(unsigned& attackDamage);
	static void removeShield(const ShieldType&);
	static void setShield(const ShieldType&, ShieldHandler*, const unsigned&);
	static void addShield(const ShieldType&, const unsigned&);
private:
	static ShieldHandler* innerShield;
	static ShieldHandler* middleShield;
	static ShieldHandler* outerShield;
};
ShieldHandler* ShieldManager::innerShield = nullptr;
ShieldHandler* ShieldManager::middleShield = nullptr;
ShieldHandler* ShieldManager::outerShield = nullptr;



// Handler
class ShieldHandler {
public:
	ShieldHandler(ShieldHandler * inputSucessor, const unsigned & inputDefensePower) : sucessor(inputSucessor), defensePower(inputDefensePower) { }
	virtual ~ShieldHandler() = default;

	void setHandler(ShieldHandler* inputSucessor, const unsigned& inputDefensePower) {
		sucessor = inputSucessor;
		defensePower = inputDefensePower;
	}
	virtual void defense(unsigned& attackPower) {
		playDestroyAnimation();
		attackPower -= defensePower;
		if (sucessor != nullptr)
			sucessor->defense(attackPower);
	}
	virtual void playDestroyAnimation() = 0;
protected:
	ShieldHandler* sucessor;
	unsigned defensePower;
};


// ConcreteHandler
class InnerShield : public ShieldHandler {
public:
	InnerShield(ShieldHandler* inputSucessor, const unsigned& inputDefensePower) : ShieldHandler(inputSucessor, inputDefensePower) { 
		std::cout << "Inner Shield object is created" << std::endl;
	}
	~InnerShield() {
		std::cout << "Inner Shield object is actually destroyed" << std::endl;
	}

	void defense(unsigned& attackPower) override {
		if (defensePower <= attackPower) {
			ShieldHandler::defense(attackPower);
			ShieldManager::removeShield(ShieldType::Inner);
		}
		else
			defenseInnerShield(attackPower);
	}
	void playDestroyAnimation() override {
		playInnerShieldDestroyAnimation();
	}
private:
	void playInnerShieldDestroyAnimation() {
		std::cout << "Inner shield is destroyed" << std::endl;
	}
	void defenseInnerShield(unsigned attackPower) {
		defensePower -= attackPower;
		// some executions
		std::cout << "Inner shield sucessfully defended the attack" << std::endl;
	}
};
class MiddleShield : public ShieldHandler {
public:
	MiddleShield(ShieldHandler* inputSucessor, const unsigned& inputDefensePower) : ShieldHandler(inputSucessor, inputDefensePower) {
		std::cout << "Middle Shield object is created" << std::endl;
	}
	~MiddleShield() {
		std::cout << "Middle Shield object is actually destroyed" << std::endl;
	}

	void defense(unsigned& attackPower) override {
		if (defensePower <= attackPower) {
			ShieldHandler::defense(attackPower);
			ShieldManager::removeShield(ShieldType::Middle);
		}
		else
			defenseMiddleShield(attackPower);
	}
	void playDestroyAnimation() override {
		playMiddleShieldDestroyAnimation();
	}
private:
	void playMiddleShieldDestroyAnimation() {
		std::cout << "Middle shield is destroyed" << std::endl;
	}
	void defenseMiddleShield(unsigned attackPower) {
		defensePower -= attackPower;
		// some executions
		std::cout << "Middle shield sucessfully defended the attack" << std::endl;
	}
};
class OuterShield : public ShieldHandler {
public:
	OuterShield(ShieldHandler* inputSucessor, const unsigned& inputDefensePower) : ShieldHandler(inputSucessor, inputDefensePower) {
		std::cout << "Outer Shield object is created" << std::endl;
	}
	~OuterShield() {
		std::cout << "Outer Shield object is actually destroyed" << std::endl;
	}

	void defense(unsigned& attackPower) override {
		if (defensePower <= attackPower) {
			ShieldHandler::defense(attackPower);
			ShieldManager::removeShield(ShieldType::Outer);
		}
		else
			defenseOuterShield(attackPower);
	}
	void playDestroyAnimation() override {
		playOuterShieldDestroyAnimation();
	}
private:
	void playOuterShieldDestroyAnimation() {
		std::cout << "Outer shield is destroyed" << std::endl;
	}
	void defenseOuterShield(unsigned attackPower) {
		defensePower -= attackPower;
		// some executions
		std::cout << "Outer shield sucessfully defended the attack" << std::endl;
	}
};




// this is not related to Chain of Responsibility pattern
void ShieldManager::defense(unsigned& attackDamage) {
	if (outerShield != nullptr)
		outerShield->defense(attackDamage);
	else if (middleShield != nullptr)
		middleShield->defense(attackDamage);
	else if (innerShield != nullptr)
		innerShield->defense(attackDamage);
	else
		std::cout << "defeat" << std::endl;
}
void ShieldManager::removeShield(const ShieldType& shieldType) {
	switch (shieldType) {
	case ShieldType::Inner:
		delete dynamic_cast<InnerShield*>(innerShield);
		innerShield = nullptr;
		break;
	case ShieldType::Middle:
		delete dynamic_cast<MiddleShield*>(middleShield);
		middleShield = nullptr;
		break;
	case ShieldType::Outer:
		delete dynamic_cast<OuterShield*>(outerShield);
		outerShield = nullptr;
		break;
	}
}
void ShieldManager::setShield(const ShieldType& shieldType, ShieldHandler* sucessor, const unsigned& defensePower) {
	switch (shieldType) {
	case ShieldType::Inner:
		innerShield->setHandler(sucessor, defensePower);
		break;
	case ShieldType::Middle:
		middleShield->setHandler(sucessor, defensePower);
		break;
	case ShieldType::Outer:
		outerShield->setHandler(sucessor, defensePower);
		break;
	}
}
void ShieldManager::addShield(const ShieldType& shieldType, const unsigned& inputDefensePower) {
	switch (shieldType) {
	case ShieldType::Inner:
		if (innerShield == nullptr)
			innerShield = new InnerShield(nullptr, inputDefensePower);
		else
			std::cout << "Inner shield is already installed" << std::endl;
		break;
	case ShieldType::Middle:
		if (middleShield == nullptr)
			middleShield = new MiddleShield(innerShield, inputDefensePower);
		else
			std::cout << "Middle shield is already installed" << std::endl;
		break;
	case ShieldType::Outer:
		if (outerShield == nullptr)
			outerShield = new OuterShield(middleShield, inputDefensePower);
		else
			std::cout << "Outer shield is already installed" << std::endl;
		break;
	}
}




// some codes
ShieldManager::addShield(ShieldType::Inner, 10);
ShieldManager::addShield(ShieldType::Inner, 20);
ShieldManager::addShield(ShieldType::Middle, 30);
ShieldManager::addShield(ShieldType::Outer, 60);
std::cout << "\n\n" << std::endl;


unsigned canonDamage = 70;
std::cout << "Canon damage: " << canonDamage << std::endl;
ShieldManager::defense(canonDamage);
std::cout << "\n\n" << std::endl;


for (unsigned curType = 0, endType = static_cast<unsigned>(ShieldType::LastShieldType); curType != endType; curType++)
    ShieldManager::removeShield(static_cast<ShieldType>(curType));

/*
print result
Inner Shield object is created
Inner shield is already installed
Middle Shield object is created
Outer Shield object is created



Canon damage: 70
Outer shield is destroyed
Middle shield sucessfully defended the attack
Outer Shield object is actually destroyed



Inner Shield object is actually destroyed
Middle Shield object is actually destroyed
*/
```

### Implementation
Here are implementation issues to consider in Chain of Responsibility:
1. *Implementing the successor chain*
    * There are two possible ways to implement the successor chain:
        + (a) Define new links (usually in the Handler, but ConcreteHandlers could define them instead).
        + (b) Use existing links.
    * Using existing links works well when the links support the chain you need. It saves you from defining links explicitly, and it saves space.
        + But if the structure doesn't reflect the chain of responsibility your application requires, then you'll have to define redundant links.
2. *Connecting successors*
    * If there are no preexisting references for defining a chain, then you'll have to introduce them yourself.
        + In that case, the Handler not only defines the interface for the requests but usually maintains the successor as well.
            - That lets the handler provide a default implementation of HandleRequest that forwards the request to the successor (if any).
        + If a ConcreteHandler subclass isn't interested in the request, it doesn't have to override the forwarding operation, since its default implementation forwards unconditionally.
    * Here's a HelpHandler base class that maintains a successor link:
        ```c++
        class HelpHandler { 
        public:
            HelpHandler(HelpHandler* s) : successor(s) { }
            virtual void handleHelp();
        private:
            HelpHandler* successor;
        };

        void HelpHandler::handleHelp() override {
            if(successor) 
                successor->handleHelp();
        }
        ```
3. *Representing requests*
    * Different options are available for representing requests.
    * In the **simplest** form, the request is a **hard-coded** operation invocation.
        + This is convenient and safe, but you can forward only the fixed set of requests that the Handler class defines.
    * An alternative is to use a single handler **function** that takes a **request code**(e.g., an integer constant or a string) as parameter.
        + This supports an open-ended set of requests.
        + The only requirement is that the sender and receiver agree on how the request should be encoded.
        + This approach is more flexible, but it requires conditional statements for dispatching the request based on its code.
            - Moreover, there's no type-safe way to pass parameters, so they must be packed and unpacked manually.
        + Obviously this is less safe than invoking an operation directly.
    * To address the parameter-passing problem, we can use separate **request objects** that bundle request parameters.
        + A **`Request` class** can represent requests explicitly, and new kinds of requests can be defined by subclassing.
        + Subclasses can define different parameters.
        + Handlers must know the kind of request (that is, Which `Request` subclass they're using) to access these parameters.
        + To identify the request, `Request` can define an accessor function that returns an identifier for the class.
            - Alternatively, the receiver can use run-time type information if the implementation languages supports it.
        + Here is a sketch of a dispatch function that uses request objects to identify requests. A `getKind` operation defined in the base `Request` class identifies the kind of request:
            ```c++
            void Handler::handleRequest(Request* theRequest) {
                switch(theRequest->getKind()) {
                case Help:
                    // cast argument to appropriate type
                    handleHelp((HelpRequest*) theRequest);
                    break;
                case Print:
                    handlePrint((PrintRequest*) theRequest);
                    //...
                    break;
                default:
                    //...
                    break;
                } 
            }
            ```
        + Subclasses can extend the dispatch by overriding `handleRequest()`. 
        + The subclass handles only the requests in which it's interested;
            - other requests are forwarded to the parent class.
            - In this way, subclasses effectively extend (rather than override) the HandleRequest operation. 
        + For example, here's how an ExtendedHandler subclass extends Handler's version of HandleRequest:
            ```c++
            class ExtendedHandler : public Handler {
            public:
                virtual void handleRequest(Request* theRequest);
                //...
            };

            void ExtendedHandler::handleRequest(Request* theRequest) {
                switch(theRequest->getKind()) {
                case Preview:
                    // handle the Preview request
                    break;
                default:
                    // let Handler handle other requests
                    Handler::handleRequest(theRequest);
                } 
            }
            ```

### Related Patterns
- Chain of Responsibility is often applied in conjunction with **Composite**.
    * There, a component's parent can act as its successor.


## Consequences
Chain of Responsibility has the following benefits and liabilities:
1. *Reduced coupling*
    * The pattern frees an object from knowing which other object handles a request.
        + Both the receiver and the sender have no explicit knowledge of each other, and an object in the chain doesn't have to know about the chain's structure.
    * As a result, Chain of Responsibility can **simplify** object interconnections. 
    * Instead of objects maintaining references to all candidate receivers, they keep a **single reference** to their successor.
2. *Added flexibility in assigning responsibilities to objects*
    * Chain of Responsibility gives you added flexibility in distributing responsibilities among objects.
    * You can **add** or **change** responsibilities for **handling a request** by adding to or otherwise changing the chain at **run-time**.
        + You can combine this with **subclassing** to specialize handlers **statically**.
3. *Receipt isn't guaranteed*
    * Since a request has **no explicit receiver**, there's **no guarantee** it'll be **handled**
        + the request can fall off the end of the chain without ever being handled.
    * A request **can** also go **unhandled** when the **chain** is not **configured properly**.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}