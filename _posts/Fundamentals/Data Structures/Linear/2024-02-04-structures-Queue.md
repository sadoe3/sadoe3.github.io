---
title: "Data Structures : Queue"

categories:
    - data-structures

tags:
    - [Data Structures, C++, Linear Data Structures, Queue]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-04
---

# Queue

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
Queues are typical examples of **linear** data structures

### FIFO
The core concept of queues is that the first element added to the queue is the first element to be removed.
- this is called **First-In-First-Out**

### Double-Ended-Queue
Double-ended-queue (**deque**) is similar to the normal queue
- the difference is that deque performs the core operations on **both** of the **first** element and the **last** element


## Core Operations

### push()
- adds the new element as the **last element**

### pop()
- removes the **first element**

### get()
- returns the **first element**


## Implementation

### Queue
```c++
template <typename Type>
class Queue {
public:
	Queue() : elements(nullptr), startIndex(0), count(0), size(4) {
		elements = new Type[size];
	}
	~Queue() {
		delete[] elements;
	}
	
	void push(const Type& data);
	Type pop() {
		if (startIndex == count)
			throw std::exception("trying to delete empty collection");
		else {
			startIndex++;
			return elements[startIndex-1];
		}
	}
	Type& get() {
		if (startIndex == count)
			throw std::exception("trying to access empty collection");
		else
			return elements[startIndex];
	}
private:
	Type* elements;
	unsigned startIndex;
	unsigned count;
	unsigned size;
};
template <typename Type>
void Queue<Type>::push(const Type& data) {
	elements[count] = data;
	count++;

	if (count == size) {
		size *= 2;
		Type* newElements = new Type[size];
		for (unsigned currentCount = 0; currentCount + startIndex < count; currentCount++)
			newElements[currentCount] = elements[currentCount + startIndex];
		delete[] elements;
		elements = newElements;

		count -= startIndex;
		startIndex = 0;
	}
}


// some codes
Queue<int> queue;

try {
	queue.get();
}
catch (std::exception e) {
	std::cerr << e.what() << std::endl;
}
try {
	queue.pop();
}
catch (std::exception e) {
	std::cerr << e.what() << std::endl;
}

queue.push(3);
queue.push(1);
queue.push(5);
std::cout << "first element: " << queue.get() << std::endl;

queue.get() = 9;
std::cout << "first element: " << queue.get() << std::endl;

std::cout << "removed element: " << queue.pop() << std::endl;
queue.push(7);
queue.push(1);
queue.push(2);
queue.push(3);
std::cout << "first element: " << queue.get() << std::endl;



/*
print result
trying to access empty collection
trying to delete empty collection
first element: 3
first element: 9
removed element: 9
first element: 1
*/
```

### Double-Ended-Queue
```c++
template <typename Type>
class Deque {
public:
	Deque() : elements(nullptr), count(0), size(4) {
		elements = new Type[size];
	}
	~Deque() {
		delete[] elements;
	}
	
	void pushFront(const Type& data);
	void pushBack(const Type& data);
	Type popFront() {
		Type copiedFirstElement = elements[0];
		
		count--;
		for (unsigned currentCount = 0; currentCount < count; currentCount++)
			elements[currentCount] = elements[currentCount + 1];

		return copiedFirstElement;
	}
	Type popBack() {
		if (count > 0) {
			count--;
			return elements[count];
		}
		else
			throw std::exception("trying to delete empty collection");
	}
	Type& getFront() {
		if (count > 0)
			return elements[0];
		else
			throw std::exception("trying to access empty collection");
	}
	Type& getBack() {
		if (count > 0)
			return elements[count-1];
		else
			throw std::exception("trying to access empty collection");
	}
	// print() is not necessary for deque; this exists for test purpose only
	void print() {
		for (unsigned currentCount = 0; currentCount < count; currentCount++)
			std::cout << elements[currentCount] << " ";
		std::cout << std::endl;
	}
private:
	Type* elements;
	unsigned count;
	unsigned size;
};
template <typename Type>
void Deque<Type>::pushFront(const Type& data) {
	count++;

	if (count == size) {
		size *= 2;
		Type* newElements = new Type[size];
		for (unsigned currentCount = 0; currentCount < count; currentCount++)
			newElements[currentCount+1] = elements[currentCount];
		delete[] elements;
		elements = newElements;
	}
	else {
		for (unsigned currentCount = count-1; currentCount > 0; currentCount--)
			elements[currentCount] = elements[currentCount-1];
	}
	elements[0] = data;
}
template <typename Type>
void Deque<Type>::pushBack(const Type& data) {
	elements[count] = data;
	count++;

	if (count == size) {
		size *= 2;
		Type* newElements = new Type[size];
		for (unsigned currentCount = 0; currentCount < count; currentCount++)
			newElements[currentCount] = elements[currentCount];
		delete[] elements;
		elements = newElements;
	}
}



// some codes
Deque<int> deque;

try {
	deque.getFront();
}
catch (std::exception e) {
	std::cerr << e.what() << std::endl;
}
try {
	deque.popBack();
}
catch (std::exception e) {
	std::cerr << e.what() << std::endl;
}

deque.pushBack(3);
deque.pushFront(1);
deque.pushFront(5);
std::cout << "last element: " << deque.getBack() << std::endl;

deque.getBack() = 9;
std::cout << "last element: " << deque.getBack() << std::endl;

std::cout << "removed element: " << deque.popBack() << std::endl;
std::cout << "last element: " << deque.getBack() << std::endl;
deque.pushBack(7);
deque.pushFront(1);
deque.pushFront(2);
deque.pushBack(3);
deque.popFront();
deque.print();
std::cout << "first element: " << deque.getFront() << std::endl;



/*
print result
trying to access empty collection
trying to delete empty collection
last element: 3
last element: 9
removed element: 9
last element: 1
1 5 1 7 3
first element: 1
*/
```

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}