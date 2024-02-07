---
title: "Data Structures : Binary Search Tree"

categories:
    - data-structures

tags:
    - [Data Structures, C++, Non-linear Data Structures, Binary Search Tree]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-07
---

# Binary Search Tree

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
Binary search trees are typical examples of **non-linear** data structures

### Binary Search Tree
**Binary search trees** are implemented based on the [**binary tree**](https://sadoe3.github.io/data-structures/structures-Tree/#binary-tree)
- the difference is that
- in binary trees
    * each node has an **unique** value
        + which means that there's **no duplicate** node in binary trees
    * newly added node would be **left** child node
        + if its value is **smaller**
    * otherwise (if it's **greater** than the target node), it would be **right** child node

### Traversal
Because binary search trees are implemented based on the binary tree, the **traversal** machnisms are same as the ones from the binary tree.
- in this post, **preorder** traversal is selected

### Removing a child node
When you remove the data based on the given value, follow the steps below: 
1. find the **target node** which has that given value
2. create a **temporary collection** which copies the data of the subree of which root node is that **target node** by using **preorder** traversal
3. delete that subtree 
4. add the rest data from the temporary collection to the original root node
    * which means use the index from `1` to the last one
    * because `0` would be the target data to remove


## Core Operations

### add()
- adds new node which has the given value to the binary search tree
- returns `true` if the addition succeeded
    * which means there's no duplicate node
- oterwise, returns `false`

### remove()
- removes the target node which has the given value
- follows the steps mentioned above
- returns the pointer to the current root node
    * which means that if you try to remove root node, then it would return the pointer which points to the new root node

### get()
- returns the pointer to the target node which has the given value
- returns `nullptr` if there's no target node


## Implementation

### Data
```c++
struct Data {
	int value;
	/*other data*/
};
bool operator==(const Data& lhs, const Data& rhs) {
	return lhs.value == rhs.value;
}
bool operator<(const Data& lhs, const Data& rhs) {
	return lhs.value < rhs.value;
}
bool operator>(const Data& lhs, const Data& rhs) {
	return lhs.value > rhs.value;
}
```

### Node
```c++
class Node {
	// Queue is implemented by using the previous post regarding queue
	friend void getDataCollectionPreorder(Queue<Data>&, Node*);
	friend void printBSTPreorder(Node* currentNode);
public:
	Node(const Data& inputData) : data(inputData), leftChild(nullptr), rightChild(nullptr) { }
	~Node() {
		delete leftChild;
		delete rightChild;
		std::cout << "delete " << data.value << std::endl;
	}

	bool add(const Data&);
	Node* remove(const Data&);
	Node* get(const Data&);
private:
	Node* getParent(const Data&);

	Data data;

	Node* leftChild;
	Node* rightChild;
};
bool Node::add(const Data& inputData) {
	if (data == inputData)
		return false;

	if (inputData < data) {
		if (leftChild == nullptr) {
			leftChild = new Node(inputData);
			return true;
		}
		return leftChild->add(inputData);
	}
	else {
		if (rightChild == nullptr) {
			rightChild = new Node(inputData);
			return true;
		}
		return rightChild->add(inputData);
	}
}
Node* Node::remove(const Data& targetData) {
	auto targetParentNode = getParent(targetData);
	Queue<Data> temporaryCollection;

	if (targetParentNode == nullptr) {
		if (data == targetData) {
			// attempt to remove root node
			getDataCollectionPreorder(temporaryCollection, this);
			temporaryCollection.pop();
			Node* newRootNode = new Node({ temporaryCollection.pop() });
			while (true) {
				try {
					auto currentData = temporaryCollection.pop();
					newRootNode->add(currentData);
				}
				catch (std::exception e) {
					break;
				}
			}
			delete this;
			return newRootNode;
		}
		return this;
	}


	Node* targetNode;
	bool isTargetLeft;
	if (targetParentNode->leftChild->data == targetData) {
		targetNode = targetParentNode->leftChild;
		isTargetLeft = true;
	}
	else {
		targetNode = targetParentNode->rightChild;
		isTargetLeft = false;
	}

	getDataCollectionPreorder(temporaryCollection, targetNode);
	delete targetNode;

	if (isTargetLeft)
		targetParentNode->leftChild = nullptr;
	else
		targetParentNode->rightChild = nullptr;

	temporaryCollection.pop();
	while (true) {
		try {
			auto currentData = temporaryCollection.pop();
			add(currentData);
		}
		catch (std::exception e) {
			break;
		}
	}
}
Node* Node::get(const Data& targetData) {
	if (data == targetData)
		return this;

	if (leftChild != nullptr) {
		auto cachedNode = leftChild->get(targetData);
		if (cachedNode != nullptr)
			return cachedNode;
	}

	if (rightChild != nullptr) {
		auto cachedNode = rightChild->get(targetData);
		if (cachedNode != nullptr)
			return cachedNode;
	}

	return nullptr;
}
// this method is similar to get();
// however, this one finds the parent which has the target node as its child node;
// attempt to remove root node is handled in remove() method, not this one
Node* Node::getParent(const Data& targetData) {
	if (leftChild != nullptr) {
		if (leftChild->data == targetData)
			return this;

		auto cachedNode = leftChild->getParent(targetData);
		if (cachedNode != nullptr)
			return cachedNode;
	}

	if (rightChild != nullptr) {
		if (rightChild->data == targetData)
			return this;

		auto cachedNode = rightChild->getParent(targetData);
		if (cachedNode != nullptr)
			return cachedNode;
	}

	return nullptr;
}
```

### Traversal
```c++
// Queue is implemented by using the previous post regarding queue
void getDataCollectionPreorder(Queue<Data>& temporaryCollection, Node* currentNode) {
	if (currentNode != nullptr) {
		temporaryCollection.push(currentNode->data);
		getDataCollectionPreorder(temporaryCollection, currentNode->leftChild);
		getDataCollectionPreorder(temporaryCollection, currentNode->rightChild);
	}
}

void printBSTPreorder(Node* currentNode) {
	if (currentNode != nullptr) {
		std::cout << currentNode->data.value << " ";
		printBSTPreorder(currentNode->leftChild);
		printBSTPreorder(currentNode->rightChild);
	}
}
```

### Client
```c++
Node* rootNode = new Node({ 0 });

rootNode->add({ 3 });
if (rootNode->add({ 3 }) == false)
    std::cout << "duplicate checking works" << std::endl;
rootNode->add({ 1 });
rootNode->add({ -3 });
rootNode->add({ 5 });
rootNode->add({ 2 });
rootNode->add({ -7 });
rootNode->add({ -2 });

printBSTPreorder(rootNode);
std::cout << "\n" << std::endl;


auto searchResult = rootNode->get({ 5 });
if (searchResult != nullptr)
    std::cout << "searching test 1 works" << std::endl;
searchResult = rootNode->get({ 51 });
if (searchResult == nullptr)
    std::cout << "searching test 2 works" << std::endl;


rootNode = rootNode->remove({ 0 });
printBSTPreorder(rootNode);
std::cout << "\n" << std::endl;


delete rootNode;
/*
print result
duplicate checking works
0 -3 -7 -2 3 1 2 5

searching test 1 works
searching test 2 works
delete -7
delete -2
delete -3
delete 2
delete 1
delete 5
delete 3
delete 0
-3 -7 -2 3 1 2 5

delete -7
delete 2
delete 1
delete 5
delete 3
delete -2
delete -3
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}