---
title: "Data Structures : Tree"

categories:
    - data-structures

tags:
    - [Data Structures, C++, Non-linear Data Structures, Tree, Binary Tree]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-06
---

# Tree

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
Trees are typical examples of **non-linear** data structures

### Tree
**Trees** are implemented based on the [**graph**](https://sadoe3.github.io/data-structures/structures-Graph/).
- the difference between the tree and the graph is that
    * tree has a **starting point**
    * therefore, tree contains a hierarchy
- in the perspective of trees, an element is regarded as a **Node**
    * the starting point is referred to as a **root** node
    * a node can be a **parent** node ** which may have **child** node(s)
    * if the node doest **not have any child** node, then it is called a **leaf** (terminal) node
- the **degree** of the node is
    * the number of its **direct child** node(s)
- the **height** of the tree is
    * the maximum **level** of the tree
    * ![LevelsOfTree.pdf](https://github.com/sadoe3/sadoe3.github.io/files/14176233/LevelsOfTree.pdf){: width="50%" height="50%"}

### Binary Tree
**Binary trees** are implemented based on the **tree**
- the difference is that
    * for binary tree, the maximum number of child nodes is `2`
- there are several types of binary trees
    * **Skewed Binary Tree**
        + **only** left or right child nodes exist 
    * **Full Binary Tree**
        + it's **not possible to add new node** unless the tree increase its level by making the leaf node have the child node
    * **Complete Binary Tree**
        + a binary tree in which every level, **except possibly the last**, is **completely filled**, and all nodes in the last level are as far left as possible
        + you can think of this type of this as the **previous status** of the **full binary tree**


## Core Operations
Because trees are implemented based on the graph, the core operations is similar to the graph. Moreover this post mainly covers regarding the binary tree. 

### setLeftChild(), setRightChild()
- are based on the `connect()` method from the graph
- make the target node have the given node as their left or right child node
- copy the previous left or right node before setting new one and return that copy 

### resetLeftChild(), resetRightChild()
- set the target node's left or right child node as `nullptr`
- copy the left or right child node before resetting and returns that copy 


## Binary Tree Traversal
Binary traversal is implemented through the **non-member function**.
- in order to undrestand the traversal of the binary tree
    * you have to understand the following concepts:
        + `D` : to read the data of the current node
        + `L` : to move to its left child node
        + `R` : to move to its right child node
- there are 3 ways to traverse the binary tree which have the difference order of those 3 concepts above
    * Preorder Traversal
        + `D` - `L` - `R`
    * Inorder Traversal
        + `L` - `D` - `R`
    * Postorder Traversal
        + `L` - `R` - `D`


## Implementation
Trees can be implemented through an **array** or a **linked list**.
- in this post, linked lists are used
- a method using an array is covered in the post regarding the [**heap**](https://sadoe3.github.io/data-structures/structures-Heap/)

### Binary Tree
```c++
struct Data {
	int value;
	/*other data*/
};

class Node {
public:
	Node(const Data& inputData) : data(inputData), leftChild(nullptr), rightChild(nullptr) { }
	~Node() {
		delete leftChild;
		delete rightChild;
		std::cout << data.value << " is removed" << std::endl;
	}

	Node* setLeftChild(Node* inputLeftChild) {
		Node* cachedChild = leftChild;
		leftChild = inputLeftChild;
		return cachedChild;
	}
	Node* setRightChild(Node* inputRightChild) {
		Node* cachedChild = rightChild;
		rightChild = inputRightChild;
		return cachedChild;
	}

	Node* resetLeftChild() {
		Node* cachedChild = leftChild;
		leftChild = nullptr;
		return cachedChild;
	}
	Node* resetRightChild() {
		Node* cachedChild = rightChild;
		rightChild = nullptr;
		return cachedChild;
	}
private:
	Data data;

	Node* leftChild;
	Node* rightChild;
};
```
```c++
Node* rootNode = new Node({ 0 });

Node* leftSubTree = new Node({ -1 });
Node* rightSubTree = new Node({ 1 });

leftSubTree->setLeftChild(new Node({ 10 }));
leftSubTree->setRightChild(new Node({ 20 }));
rightSubTree->setLeftChild(new Node({ -10 }));
rightSubTree->setRightChild(new Node({ -20 }));

rootNode->setLeftChild(leftSubTree);
rootNode->setRightChild(rightSubTree);

delete rootNode;
/*
print result
10 is removed
20 is removed
-1 is removed
-10 is removed
-20 is removed
1 is removed
0 is removed
*/
```

### Preorder Traversal
`D` - `L` - `R`
```c++
class Node {
	friend void traversePreorder(Node*);
    // other definitions
};
```
```c++
void traversePreorder(Node* currentNode) {
    if(currentNode != nullptr) {
        std::cout << currentNode->data.value << " ";    // this code can be changed based on the context
        traversePreorder(currentNode->leftChild);
        traversePreorder(currentNode->rightChild);
    }
}
```
```c++
Node* rootNode = new Node({ 0 });

Node* leftSubTree = new Node({ -1 });
Node* rightSubTree = new Node({ 1 });

leftSubTree->setLeftChild(new Node({ 10 }));
leftSubTree->setRightChild(new Node({ 20 }));
rightSubTree->setLeftChild(new Node({ -10 }));
rightSubTree->setRightChild(new Node({ -20 }));

rootNode->setLeftChild(leftSubTree);
rootNode->setRightChild(rightSubTree);


traversePreorder(rootNode);


delete rootNode;
/*
print result
0 -1 10 20 1 -10 -20
*/
```

### Inorder Traversal
`L` - `D` - `R`
```c++
class Node {
	friend void traverseInorder(Node*);
    // other definitions
};
```
```c++
void traverseInorder(Node* currentNode) {
    if(currentNode != nullptr) {
        traverseInorder(currentNode->leftChild);
        std::cout << currentNode->data.value << " ";    // this code can be changed based on the context
        traverseInorder(currentNode->rightChild);
    }
}
```
```c++
Node* rootNode = new Node({ 0 });

Node* leftSubTree = new Node({ -1 });
Node* rightSubTree = new Node({ 1 });

leftSubTree->setLeftChild(new Node({ 10 }));
leftSubTree->setRightChild(new Node({ 20 }));
rightSubTree->setLeftChild(new Node({ -10 }));
rightSubTree->setRightChild(new Node({ -20 }));

rootNode->setLeftChild(leftSubTree);
rootNode->setRightChild(rightSubTree);


traverseInorder(rootNode);


delete rootNode;
/*
print result
10 -1 20 0 -10 1 -20
*/
```

### Postorder Traversal
`L` - `R` - `D`
```c++
class Node {
	friend void traversePostorder(Node*);
    // other definitions
};
```
```c++
void traversePostorder(Node* currentNode) {
    if(currentNode != nullptr) {
        traversePostorder(currentNode->leftChild);
        traversePostorder(currentNode->rightChild);
        std::cout << currentNode->data.value << " ";    // this code can be changed based on the context
    }
}
```
```c++
Node* rootNode = new Node({ 0 });

Node* leftSubTree = new Node({ -1 });
Node* rightSubTree = new Node({ 1 });

leftSubTree->setLeftChild(new Node({ 10 }));
leftSubTree->setRightChild(new Node({ 20 }));
rightSubTree->setLeftChild(new Node({ -10 }));
rightSubTree->setRightChild(new Node({ -20 }));

rootNode->setLeftChild(leftSubTree);
rootNode->setRightChild(rightSubTree);


traversePostorder(rootNode);


delete rootNode;
/*
print result
10 20 -1 -10 -20 1 0
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}