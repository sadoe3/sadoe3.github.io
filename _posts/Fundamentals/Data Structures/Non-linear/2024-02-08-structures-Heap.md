---
title: "Data Structures : Heap"

categories:
    - data-structures

tags:
    - [Data Structures, C++, Non-linear Data Structures, Heap]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-08
---

# Heap

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
Heaps are typical examples of **non-linear** data structures

### Heap
**Heap**s are implemented based on the [**binary trees**](https://sadoe3.github.io/data-structures/structures-Tree/#binary-tree)
- the difference is that
    * in heaps
        + whenever new node is **added** or existing node is **removed**
		+ the **root** node should be 
	        - the **maximum** or the **minimum** node
- if it has the maximum value
	* then that heap is called as **max heap**
- otherwise it’s called as **min heap**

### Max Heap and Min Heap
- for **max** heap
    * the value of the parent node `>=` the values of its child nodes
- for **min** heap
	* the value of the parent node `<=` the values of its child nodes


## Core Operations

### add()
This method follows the steps listed below 
1.	add the new node at the position of the last index
2.	compare it with its direct parent node
    * if the relationship is not proper
        + swap the parent and the new node
    * otherwise
        + the process is done
3.	repeat step 2 until the process is done

### remove()
The important point of remove() method is that only root node can be deleted
- this method follows the steps listed below
1.	swap the root node and the last element
2.	remove the root node at the position of the last index
3.	compare the last element at the position of the first index with its direct child node
    * if the relationship is not proper,
        + swap them
    * otherwise
        + the process is ended
4.	repeat step 3 until the process is done

### getRootNode()
- returns the pointer to the root node


## Implementation
Heaps can be implemented with the data structure that supports the **random access with the given index**
- the point is that
    * the **index of the root** node should be `1`
        + so that the index of the parent node `=` index / 2
        + and the indices of its child nodes `=` index * 2, (index * 2) + 1

### Collection
```c++
template <typename Type>
struct Collection {
	Collection() : elements(nullptr), count(1), size(4) {
		elements = new Type[size];
	}
	~Collection() {
		delete[] elements;
	}

	void pushBack(const Type& data);
	Type popBack() {
		if (count > 1) {
			count--;
			return elements[count];
		}
		else
			throw std::exception("trying to delete empty collection");
	}
	Type* getPointerToRootNode() {
		if (count > 1)
			return &elements[1];
		else
			return nullptr;
	}

	Type* elements;
	unsigned count;
	unsigned size;
};
```
```c++
template <typename Type>
void Collection<Type>::pushBack(const Type& data) {
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
```

### Min Heap
```c++
template <typename Type>
class MinHeap {
public:
	MinHeap() : collection() { }

	void add(const Type& inputData);
	void remove();
	Type* getRootNode() {
		return collection.getPointerToRootNode();
	}

private:
	Collection<Type> collection;
};
```
```c++
template <typename Type>
void MinHeap<Type>::add(const Type& inputData) {
	collection.pushBack(inputData);
	Type* elements = collection.elements;
	unsigned currentIndex = collection.count - 1;
	Type cachedData;

	while (currentIndex != 1) {
		if (elements[currentIndex / 2] > elements[currentIndex]) {
			cachedData = elements[currentIndex / 2];
			elements[currentIndex / 2] = elements[currentIndex];
			elements[currentIndex] = cachedData;

			currentIndex /= 2;
		}
		else
			break;
	}
}
```
```c++
template <typename Type>
void MinHeap<Type>::remove() {
    if (collection.count > 1) {
        Type* elements = collection.elements;
        Type cachedData = elements[1];
        unsigned currentIndex = collection.count - 1;
        elements[1] = elements[currentIndex];
        elements[currentIndex] = cachedData;

        collection.popBack();
        currentIndex = 1;
        while (currentIndex < collection.count) {
            if (elements[currentIndex * 2] < elements[currentIndex * 2 + 1]) {
                if (currentIndex * 2 < collection.count && elements[currentIndex] > elements[currentIndex * 2]) {
                    cachedData = elements[currentIndex];
                    elements[currentIndex] = elements[currentIndex * 2];
                    elements[currentIndex * 2] = cachedData;

                    currentIndex *= 2;
                }
                else
                    break;
            }
            else {
                if (currentIndex * 2 + 1 < collection.count && elements[currentIndex] > elements[currentIndex * 2 + 1]) {
                    cachedData = elements[currentIndex];
                    elements[currentIndex] = elements[currentIndex * 2 + 1];
                    elements[currentIndex * 2 + 1] = cachedData;

                    currentIndex = currentIndex * 2 + 1;
                }
                else
                    break;
            }
        }

        if (collection.count == 3) {
            if (elements[1] > elements[2]) {
                cachedData = elements[1];
                elements[1] = elements[2];
                elements[2] = cachedData;
            }
        }
    }
}
```
```c++
MinHeap<int> heap;

if (heap.getRootNode() == nullptr)
    std::cout << "null checking works" << std::endl;

heap.add(1);
heap.add(2);
heap.add(3);
heap.add(-5);
heap.add(-2);

heap.remove();

std::cout << *(heap.getRootNode()) << std::endl;


/*
print result
null checking works
-2
*/
```

### Max Heap
```c++
template <typename Type>
class MaxHeap {
public:
	MaxHeap() : collection() { }

	void add(const Type& inputData);
	void remove();
	Type* getRootNode() {
		return collection.getPointerToRootNode();
	}

private:
	Collection<Type> collection;
};
```
```c++
template <typename Type>
void MaxHeap<Type>::add(const Type& inputData) {
	collection.pushBack(inputData);
	Type* elements = collection.elements;
	unsigned currentIndex = collection.count - 1;
	Type cachedData;

	while (currentIndex != 1) {
		if (elements[currentIndex / 2] < elements[currentIndex]) {
			cachedData = elements[currentIndex / 2];
			elements[currentIndex / 2] = elements[currentIndex];
			elements[currentIndex] = cachedData;

			currentIndex /= 2;
		}
		else
			break;
	}
}
```
```c++
template <typename Type>
void MaxHeap<Type>::remove() {
    if (collection.count > 1) {
        Type* elements = collection.elements;
        Type cachedData = elements[1];
        unsigned currentIndex = collection.count - 1;
        elements[1] = elements[currentIndex];
        elements[currentIndex] = cachedData;

        collection.popBack();
        currentIndex = 1;
        while (currentIndex < collection.count) {
            if (elements[currentIndex * 2] > elements[currentIndex * 2 + 1]) {
                if (currentIndex * 2 < collection.count && elements[currentIndex] < elements[currentIndex * 2]) {
                    cachedData = elements[currentIndex];
                    elements[currentIndex] = elements[currentIndex * 2];
                    elements[currentIndex * 2] = cachedData;

                    currentIndex *= 2;
                }
                else
                    break;
            }
            else {
                if (currentIndex * 2 + 1 < collection.count && elements[currentIndex] < elements[currentIndex * 2 + 1]) {
                    cachedData = elements[currentIndex];
                    elements[currentIndex] = elements[currentIndex * 2 + 1];
                    elements[currentIndex * 2 + 1] = cachedData;

                    currentIndex = currentIndex * 2 + 1;
                }
                else
                    break;
            }
        }

        if (collection.count == 3) {
            if (elements[1] < elements[2]) {
                cachedData = elements[1];
                elements[1] = elements[2];
                elements[2] = cachedData;
            }
        }
    }
}
```
```c++
MaxHeap<int> heap;

if (heap.getRootNode() == nullptr)
    std::cout << "null checking works" << std::endl;

heap.add(1);
heap.add(2);
heap.add(3);
heap.add(-5);
heap.add(-2);

heap.remove();

std::cout << *(heap.getRootNode()) << std::endl;


/*
print result
null checking works
2
*/
```



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}