---
title: "Algorithms : Hashing"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Search, Hash, Hashing]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-19
---

# Hashing

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
The algorithm for the **hashing** utilizes the **hash value** to search for the certain element

### Hash Function
A **hash function** takes the **key** of the element and returns the **hash value** based on a certain method
- there are various methods to calculate the hash value
    * in this post, the **division method** would be used to get the hash value
	    + `hash value = key % size of hash table`
- the hash value is utilized to access to the **element** in the **hash table**

### Hash Table
A **hash table** consists of **buckets** which contains one or more **slot(s)** to store elements
- the number of slots depends on how the **overflow** is handled

### Overflow
In hashing, **overflow** means a situation where **multiple elements** are being stored in the same bucket
- which means that they have the **same hash value**
    * in hashing, those elements are called as **synonyms**
- in this post, 2 ways which handle the **overflow** would be covered
- **chaining**
	* through this method, **hash table** is the **array** of **[LinkedList](link to linked list)s of the elements**
	    + the **number of slots** = `one or more`
	* this method handles the **overflow** by
	    + **inserting** the element to the proper **linked list** (bucket)
- **linear probing**
    * through this method, **hash table** is the **array** of the **elements**
	    + the **number of slots** = `one`
	* this method handles the **overflow** by
	    + **increasing** the hash value like this :
            ```c++
            while(hashTable[hashValue] != defaultElement && hashValue < sizeOfHashTable)
                hashValue++;
            ```
        + then,
            ```c++
            if(hashValue < sizeOfHashTable)
                hashTable[hashValue] = currentElement;
            ```
	* note that every slot in this hash table is initialized with the `defaultElement`
	    + which means that if the condition : `hashTable[hashValue] != defaultElement` is `false`
	    + then, that slot is not currently used


## Time Complexity
- Ideal: `O(1)`
- Worst: `O(s)`
    * where `s` is the number of slots of a bucket


## Implementation

### Hash Function
```c++
// this function assumes that the type of the given key is unsigned
constexpr unsigned getHashValue(const unsigned &key, const unsigned &sizeOfHashTable) {
    return (key % sizeOfHashTable);
}
```

### Linked List
```c++
struct Element {
	// key should be a unique value	
	int key;
};
```
```c++
// this code is based on the previous post regarding the linked list
class LinkedList {
public:
	virtual ~LinkedList() = default;

	virtual void add(const int&) = 0;
	virtual LinkedList* remove(const int&) = 0;
	virtual LinkedList* get(const int&) = 0;
protected:
	LinkedList(const Element &inputElement) : element(inputElement) { }
	Element element;
};
```
```c++
class SinglyLinkedList : LinkedList {
	LinkedList* get(const int& targetKey) override {
		if (element.key == targetKey)
			return this;
		else {
			if (nextNode == nullptr)
				return nullptr;
			else
				return nextNode->get(targetKey);
		}
	}
}
```
```c++
LinkedList* SinglyLinkedList::remove(const int &targetKey, bool &isRemoved) {
	if (nextNode == nullptr) {
		if (data == targetData) {
			delete this;
            isRemoved = true;
			return nullptr;
		}
		else
			return this;
	}
	else {
		if (data == targetData) {
			auto cachedNode = nextNode;
			delete this;
            isRemoved = true;
			return cachedNode;
		}
		else if (nextNode->data == targetData) {
			SinglyLinkedList* savedLinkField = nextNode->nextNode;
			delete nextNode;
            isRemoved = true;
			nextNode = savedLinkField;
			return this;
		}
		else {
			nextNode->remove(targetData);
			return this;
		}
	}
}
```

### Hash Table
```c++
constexpr int sizeOfHashTable = 30; // this can be any value

SinglyLinkedList* hashTableChaining[sizeOfHashTable];
for (unsigned currentBucket = 0; currentBucket < sizeOfHashTable; currentBucket++)
    hashTableChaining[currentBucket] = new SinglyLinkedList(Element{ 0 });

Element hashTableLinearProbing[sizeOfHashTable] = { Element{ 0 } }; // every slot is initialized with the default element
```

### Insertion
```c++
void insertElementChaining(DoubleLinkedList<Element>* hashTable[], const Element &element, const unsigned &hashValue) {
    hashTable[hashValue]->addNode(element);
}
```
```c++
bool insertElementLinearProbing(Element hashTable[], const Element &element, const unsigned &hashValue, const unsigned &sizeOfHashTable) {
    Element defaultElement;
    while(hashTable[hashValue] != defaultElement && hashValue < sizeOfHashTable)
        hashValue++;

    if(hashValue < sizeOfHashTable) {
        hashTable[hashValue] = element;
        return true;
    }
    else
        return false;
}
```

### Search
```c++
Element* searchElementChaining(SinglyLinkedList<Element>* hashTable[], const int &key, const unsigned &hashValue) {
    return hashTable[hashValue]->get(key);
}
```
```c++
Element* searchElementLinearProbing(Element hashTable[], const int &key, const unsigned &sizeOfHashTable) {
    int hashValue = getHashValue(key, sizeOfHashTable);
    while(hashValue != defaultElement && hashValue < sizeOfHashTable)
        hashValue++;
    
    if(hashValue < sizeOfHashTable)
        return &(hashTable[hashValue]);
    else
        return nullptr;
}
```

### Removal
```c++
bool removeElementChaining(SinglyLinkedList<Element>* hashTable[], const int &key, const unsigned &hashValue) {
    auto searchResult = hashTable[hashValue]->get(key);

    if(searchResult == nullptr)
        return false;
    else {
		bool isRemoved = false;
		hashTable[hashValue] = hashTable[hashValue]->remove(key, isRemoved);
		if(hashTable[hashValue] == nullptr)
			hashTable[hashValue] = new SinglyLinkedList<Element>();
		return isRemoved;
	}
}
```
```c++
bool removeElementLinearProbing(Element hashTable[], const int &key, const unsigned &sizeOfHashTable) {
    int hashValue = getHashValue(key, sizeOfHashTable);
    while(hashValue] != defaultElement && hashValue < sizeOfHashTable)
        hashValue++;
    
    if(hashValue < sizeOfHashTable) {
        Element defaultElement;
        hashTable[hashValue] = defaultElement;

        return true;
    }
    else
        return false;
}
```

### Client
```c++
// initialized hash table
// remove and reinsert another element
```
```c++
// try to search for certain element especially for the reinserted element
// print the search result
```

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}