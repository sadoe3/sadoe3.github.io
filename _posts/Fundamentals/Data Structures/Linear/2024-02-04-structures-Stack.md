---
title: "Data Structures : Stack"

categories:
    - data-structures

tags:
    - [Data Structures, C++, Linear Data Structures, Stack]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-04
---

# Stack

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
Stacks are typical examples of **linear** data structures

### LIFO
The core concept of stacks is that the last element added to the stack is the first element to be removed.
- this is called **Last-In-First-Out**


## Core Operations

### push()
- adds the new element as the **last element**

### pop()
- removes the **last element**

### get()
- returns the **element**


## Implementation

### Stack
```c++
template <typename Type>
class Stack {
public:
	Stack() : elements(nullptr), count(0), size(4) {
		elements = new Type[size];
	}
	~Stack() {
		delete[] elements;
	}
	
	void push(const Type& data);
	Type pop() {
		if (count > 0) {
			count--;			
			return elements[count];
		}
		else
			throw std::exception("trying to delete empty collection");
	}
	Type& get() {
		if (count > 0)
			return elements[count - 1];
		else
			throw std::exception("trying to access empty collection");
	}
private:
	Type* elements;
	unsigned count;
	unsigned size;
};
template <typename Type>
void Stack<Type>::push(const Type& data) {
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
Stack<int> stack;

try {
    stack.get();
}
catch (std::exception e) {
    std::cerr << e.what() << std::endl;
}
try {
    stack.pop();
}
catch (std::exception e) {
    std::cerr << e.what() << std::endl;
}

stack.push(3);
stack.push(1);
stack.push(5);
stack.push(7);
std::cout << "last element: " << stack.get() << std::endl;

stack.get() = 3;
std::cout << "last element: " << stack.get() << std::endl;

std::cout << "removed element: " << stack.pop() << std::endl;
std::cout << "last element: " << stack.get() << std::endl;



/*
print result
trying to access empty collection
trying to delete empty collection
last element: 7
last element: 3
removed element: 3
last element: 5
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}