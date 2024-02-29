---
title: "Algorithms : Tree Sort"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Sort, Tree Sort]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-18
---

# Tree Sort

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
The algorithm for the **tree sort** is based on the [**binary search tree**](https://sadoe3.github.io/data-structures/structures-BinarySearchTree/) data structure

### Order
If you traverse the binary search tree through **inorder** way
- you can sort the collection in an **ascending** order


## Time Complexity
`O(nlogn)`


## Implementation

### Binary Search Tree
[**Binary Search Tree**](https://sadoe3.github.io/data-structures/structures-BinarySearchTree/#implementation)

### Node
```c++
class Node {
public:
	~Node() {
		delete leftChild;
		std::cout << "delete " << data.value << std::endl;
		delete rightChild;
	}
    // same definition
};
```

### Vector
```c++
template <typename Type>
class Vector {
public:
	void sortTree();
// same definition
};
```

### Inorder Traversal
```c++
// Queue is implemented by using the previous post regarding queue
void getDataCollectionInorder(Queue<Data>& temporaryCollection, Node* currentNode) {
	if (currentNode != nullptr) {
		getDataCollectionInorder(temporaryCollection, currentNode->leftChild);
		temporaryCollection.push(currentNode->data);
		getDataCollectionInorder(temporaryCollection, currentNode->rightChild);
	}
}
```

### Tree Sort
```c++
// this template method assumes that it handles with integral types only
template <typename Type>
void Vector<Type>::sortTree() {
	Node* rootNode = new Node({ elements[0] });
	for (unsigned currentIndex = 1; currentIndex < count; currentIndex++)
		rootNode->add({ elements[currentIndex] });

	Queue<Data> sortedCollection;
	getDataCollectionInorder(sortedCollection, rootNode);

	for (unsigned currentIndex = 0; currentIndex < count; currentIndex++) 
		elements[currentIndex] = sortedCollection.pop().value;	

	delete rootNode;
}
```

### Client
```c++
Vector<int> collection;
collection.pushBack(1);
collection.pushBack(3);
collection.pushBack(-1);
collection.pushBack(-5);
collection.pushBack(10);
collection.pushBack(7);
collection.pushBack(-10);

printCollection(collection);

collection.sortTree();
printCollection(collection);

/*
print result
1 3 -1 -5 10 7 -10
delete -10
delete -5
delete -1
delete 1
delete 3
delete 7
delete 10
-10 -5 -1 1 3 7 10
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}