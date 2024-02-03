---
title: "Data Structures : Linked List"

categories:
    - data-structures

tags:
    - [Data Structures, C++, Linear Data Structures, Linked List]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-03
---

# Linked List

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
Linked lists are typical examples of **linear** data structures

### Node
Linked lists take their elements as a **node** which consists of 2 parts:
- **data field** (data value)
    * the primitive data of the element or
    - the operation of the element
- **link field** (link value)
    * a pointer which points to the next or previous element
    * it's default value is `nullptr`
        + and having its link field as a `nullptr` means that the next or previous element doesn't exist

### Categories
There are 3 types of linked lists
- **Singly** Linked List
    * there is a **single pointer** in the link field
        + which means there's one direction
    * it could point to the next one or the previous one
- **Doubly** Linked List
    * there are **two pointers** in the link field
        + which means there are two directions
    * they point to the previous one and the next one
- **Circular** Linked List
    * there is a **single pointer** in the link field
        + which means there's one direction
    * it could point to the next one or the previous one
    * however, it's **different** from the **Singly** Linked List
        + because **the last node points to the first node**


## Core Operations

### add()
- new node
    * points to the existing node
- existing node
    * points to the new node

### remove()
1. remove the target node
2. connect the two nodes which are previous to and next to the removed node

### get()
- sequential access only
    * random access is not supported


## Implementation

### Linked List
```c++
class LinkedList {
public:
	virtual ~LinkedList() = default;

	virtual void add(const int&) = 0;
	virtual LinkedList* remove(const int&) = 0;
	virtual LinkedList* get(const int&) = 0;
protected:
	LinkedList(const int & inputData) : data(inputData) { }
	int data;
};
```

### Singly Linked List
```c++
class SinglyLinkedList : public LinkedList {
	friend void clearCollection(SinglyLinkedList* targetNode);
	friend void printNode(const SinglyLinkedList* node);			// this code exits for test purpose only; this is not a necessary operation
public:
	SinglyLinkedList(const int& inputData) : LinkedList(inputData), nextNode(nullptr) { }
	~SinglyLinkedList() {
		std::cout << data << " is deleted" << std::endl;
	}

	void add(const int& inputData) override {
		SinglyLinkedList* newNode = new SinglyLinkedList(inputData);
		if (nextNode != nullptr)
			newNode->nextNode = nextNode;
		nextNode = newNode;
	}
	LinkedList* remove(const int& targetData) override;
	LinkedList* get(const int& targetData) override {
		if (data == targetData)
			return this;
		else {
			if (nextNode == nullptr)
				return nullptr;
			else
				return nextNode->get(targetData);
		}
	}
private:
	SinglyLinkedList* nextNode;
};
LinkedList* SinglyLinkedList::remove(const int& targetData) {
	if (nextNode == nullptr) {
		if (data == targetData) {
			delete this;
			return nullptr;
		}
		else
			return this;
	}
	else {
		if (data == targetData) {
			auto cachedNode = nextNode;
			delete this;
			return cachedNode;
		}
		else if (nextNode->data == targetData) {
			SinglyLinkedList* savedLinkField = nextNode->nextNode;
			delete nextNode;
			nextNode = savedLinkField;
			return this;
		}
		else {
			nextNode->remove(targetData);
			return this;
		}
	}
}


void clearCollection(SinglyLinkedList* targetNode) {
	if (targetNode->nextNode != nullptr)
		clearCollection(targetNode->nextNode);

	delete targetNode;
}


// this code exits for test purpose only; this is not a necessary operation
void printNode(const SinglyLinkedList* node) {
	std::cout << node->data << " ";
	if (node->nextNode == nullptr)
		std::cout << std::endl;
	else
		printNode(node->nextNode);
}




// some codes
LinkedList* firstNode = new SinglyLinkedList(0);

firstNode = firstNode->remove(0);
if (firstNode == nullptr)
    std::cout << "removal succeeded" << std::endl;
else
    std::cout << "removal failed" << std::endl;


firstNode = new SinglyLinkedList(3);
firstNode->add(1);
firstNode->add(7);
printNode(dynamic_cast<const SinglyLinkedList*>(firstNode));

firstNode = firstNode->remove(1);
printNode(dynamic_cast<const SinglyLinkedList*>(firstNode));


auto searchResult = firstNode->get(1);
if (searchResult == nullptr)
    std::cout << "search for 1 failed" << std::endl;
else
    std::cout << "search for 1 succeeded" << std::endl;


clearCollection(dynamic_cast<SinglyLinkedList*>(firstNode));
/*
print result
0 is deleted
removal succeeded
3 7 1
1 is deleted
3 7
search for 1 failed
7 is deleted
3 is deleted
*/
```

### Doubly Linked List
```c++
class DoublyLinkedList : public LinkedList {
	friend void clearCollection(DoublyLinkedList* targetNode);
	friend void printNode(const DoublyLinkedList* node);			// this code exits for test purpose only; this is not a necessary operation
public:
	DoublyLinkedList(const int& inputData) : LinkedList(inputData), previousNode(nullptr), nextNode(nullptr) { }
	~DoublyLinkedList() {
		std::cout << data << " is deleted" << std::endl;
	}

	void add(const int& inputData) override {
		DoublyLinkedList* newNode = new DoublyLinkedList(inputData);

		newNode->previousNode = this;
		if (nextNode != nullptr) {
			nextNode->previousNode = newNode;
			newNode->nextNode = nextNode;
		}
		nextNode = newNode;
	}
	LinkedList* remove(const int& targetData) override;
	LinkedList* get(const int& targetData) override {
		if (data == targetData)
			return this;
		else {
			if (nextNode == nullptr)
				return nullptr;
			else
				return nextNode->get(targetData);
		}
	}
private:
	DoublyLinkedList* previousNode;
	DoublyLinkedList* nextNode;
};
LinkedList* DoublyLinkedList::remove(const int& targetData) {
	if (data == targetData) {
		if (previousNode != nullptr) {
			previousNode->nextNode = nextNode;
			if (nextNode != nullptr)
				nextNode->previousNode = previousNode;

			auto cachedNode = previousNode;
			delete this;
			return cachedNode;
		}
		else if (nextNode != nullptr) {
			nextNode->previousNode = previousNode;

			auto cachedNode = nextNode;
			delete this;
			return cachedNode;
		}
		else {
			delete this;
			return nullptr;
		}
	}
	else {
		if (nextNode != nullptr)
			nextNode->remove(targetData);
		return this;
	}
}

void clearCollection(DoublyLinkedList* targetNode) {
	if (targetNode->nextNode != nullptr)
		clearCollection(targetNode->nextNode);

	delete targetNode;
}


// this code exits for test purpose only; this is not a necessary operation
void printNode(const DoublyLinkedList* node) {
	std::cout << node->data << " ";
	if (node->nextNode == nullptr)
		std::cout << std::endl;
	else
		printNode(node->nextNode);
}




// some codes
LinkedList* firstNode = new DoublyLinkedList(0);

firstNode = firstNode->remove(0);
if (firstNode == nullptr)
    std::cout << "removal succeeded" << std::endl;
else
    std::cout << "removal failed" << std::endl;


firstNode = new DoublyLinkedList(3);
firstNode->add(1);
firstNode->add(7);
printNode(dynamic_cast<const DoublyLinkedList*>(firstNode));

firstNode = firstNode->remove(1);
printNode(dynamic_cast<const DoublyLinkedList*>(firstNode));


auto searchResult = firstNode->get(1);
if (searchResult == nullptr)
    std::cout << "search for 1 failed" << std::endl;
else
    std::cout << "search for 1 succeeded" << std::endl;


clearCollection(dynamic_cast<DoublyLinkedList*>(firstNode));
/*
print result
0 is deleted
removal succeeded
3 7 1
1 is deleted
3 7
search for 1 failed
7 is deleted
3 is deleted
*/
```

### Circular Linked List
```c++
class CircularLinkedList : public LinkedList {
	friend void clearCollection(CircularLinkedList* firstNode, CircularLinkedList* targetNode);
	friend void printNode(const CircularLinkedList* firstNode, const CircularLinkedList* node);			// this code exits for test purpose only; this is not a necessary operation
public:
	CircularLinkedList(const int& inputData) : LinkedList(inputData), nextNode(this) { }
	~CircularLinkedList() {
		std::cout << data << " is deleted" << std::endl;
	}

	void add(const int& inputData) override {
		CircularLinkedList* newNode = new CircularLinkedList(inputData);

		newNode->nextNode = nextNode;
		nextNode = newNode;
	}
	LinkedList* remove(const int& targetData) override {
		return removeCircular(this, targetData);
	}
	LinkedList* get(const int& targetData) override {
		return getCircular(this, targetData);
	}
private:
	LinkedList* removeCircular(CircularLinkedList* startNode, const int& targetData);
	CircularLinkedList* getLastNode(CircularLinkedList* firstNode);
	LinkedList* getCircular(CircularLinkedList* startNode, const int& targetData);

	CircularLinkedList* nextNode;
};
LinkedList* CircularLinkedList::removeCircular(CircularLinkedList* startNode, const int& targetData) {
	if (this == nextNode) {
		if (data != targetData)
			return startNode;

		delete this;
		return nullptr;
	}
	else {
		if (data == targetData) {
			auto lastNode = getLastNode(this);
			lastNode->nextNode = nextNode;

			delete this;
			return lastNode->nextNode;
		}
		else if (nextNode->data == targetData) {
			auto savedLinkField = nextNode->nextNode;

			delete nextNode;
			nextNode = savedLinkField;
			return startNode;
		}
		else if (nextNode == startNode) {
			return startNode;
		}
		else {
			nextNode->removeCircular(startNode, targetData);
			return startNode;
		}
	}
}
CircularLinkedList* CircularLinkedList::getLastNode(CircularLinkedList* firstNode) {
	if (nextNode == firstNode)
		return this;
	else
		return nextNode->getLastNode(firstNode);
}
LinkedList* CircularLinkedList::getCircular(CircularLinkedList* startNode, const int& targetData) {
	if (nextNode->data == targetData)
		return nextNode;
	if (nextNode->nextNode == startNode)
		return nullptr;

	return nextNode->getCircular(startNode, targetData);
}



void clearCollection(CircularLinkedList* firstNode, CircularLinkedList* targetNode) {
	if (targetNode->nextNode != firstNode)
		clearCollection(firstNode, targetNode->nextNode);

	delete targetNode;
}


// this code exits for test purpose only; this is not a necessary operation
void printNode(const CircularLinkedList* firstNode, const CircularLinkedList* node) {
	std::cout << node->data << " ";
	if (node->nextNode == firstNode)
		std::cout << std::endl;
	else
		printNode(firstNode, node->nextNode);
}




// some codes
LinkedList* firstNode = new CircularLinkedList(0);

firstNode = firstNode->remove(0);
if (firstNode == nullptr)
    std::cout << "removal succeeded" << std::endl;
else
    std::cout << "removal failed" << std::endl;


firstNode = new CircularLinkedList(3);
firstNode->add(1);
firstNode->add(7);
printNode(dynamic_cast<const CircularLinkedList*>(firstNode), dynamic_cast<CircularLinkedList*>(firstNode));

firstNode = firstNode->remove(1);
printNode(dynamic_cast<const CircularLinkedList*>(firstNode), dynamic_cast<CircularLinkedList*>(firstNode));


auto searchResult = firstNode->get(1);
if (searchResult == nullptr)
    std::cout << "search for 1 failed" << std::endl;
else
    std::cout << "search for 1 succeeded" << std::endl;


clearCollection(dynamic_cast<CircularLinkedList*>(firstNode), dynamic_cast<CircularLinkedList*>(firstNode));
/*
print result
0 is deleted
removal succeeded
3 7 1
1 is deleted
3 7
search for 1 failed
7 is deleted
3 is deleted
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}