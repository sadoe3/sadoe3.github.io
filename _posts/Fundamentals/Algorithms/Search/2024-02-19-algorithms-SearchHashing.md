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
    // key cannot be 0; because it's the default value
    int key = 0;
};
```
```c++
// this code is based on the previous post regarding the linked list
class LinkedList {
public:
    virtual ~LinkedList() = default;

    virtual void add(const Element&) = 0;
    virtual LinkedList* remove(const int&, bool&) = 0;
    virtual LinkedList* get(const int&) = 0;

    // this method exists for the test purpose only
    void printInfo() {
        std::cout << element.key;
    }
protected:
    LinkedList(const Element& inputElement) : element(inputElement) { }
    Element element;
};
```
```c++
class SinglyLinkedList : public LinkedList {
public:
    SinglyLinkedList(const Element& inputElement) : LinkedList(inputElement), nextNode(nullptr) { }
    ~SinglyLinkedList() {
        delete nextNode;
    }

    void add(const Element& inputElement) override {
        SinglyLinkedList* newNode = new SinglyLinkedList(inputElement);
        if (nextNode != nullptr)
            newNode->nextNode = nextNode;
        nextNode = newNode;
    }
    LinkedList* remove(const int&, bool&) override;
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
private:
    SinglyLinkedList* nextNode;
};
```
```c++
LinkedList* SinglyLinkedList::remove(const int& targetKey, bool& isRemoved) {
    if (nextNode == nullptr) {
        if (element.key == targetKey) {
            delete this;
            isRemoved = true;
            return nullptr;
        }
        else
            return this;
    }
    else {
        if (element.key == targetKey) {
            auto cachedNode = nextNode;
            delete this;
            isRemoved = true;
            return cachedNode;
        }
        else if (nextNode->element.key == targetKey) {
            SinglyLinkedList* savedLinkField = nextNode->nextNode;
            delete nextNode;
            isRemoved = true;
            nextNode = savedLinkField;
            return this;
        }
        else {
            nextNode->remove(targetKey, isRemoved);
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
    hashTableChaining[currentBucket] = new SinglyLinkedList(Element());

Element hashTableLinearProbing[sizeOfHashTable] = { Element{ 0 } }; // every slot is initialized with the default element
```

### Insertion
```c++
void insertElementChaining(SinglyLinkedList* hashTable[], const Element& element, const unsigned& sizeOfHashTable) {
    // prevent the client from inserting the default value
    if (element.key == 0)
        return;

    hashTable[getHashValue(element.key, sizeOfHashTable)]->add(element);
}
```
```c++
bool insertElementLinearProbing(Element hashTable[], const Element& element, const unsigned& sizeOfHashTable) {
    Element defaultElement;
    unsigned hashValue = getHashValue(element.key, sizeOfHashTable);
    while (hashTable[hashValue].key != defaultElement.key && hashValue < sizeOfHashTable)
        hashValue++;

    if (hashValue < sizeOfHashTable) {
        hashTable[hashValue] = element;
        return true;
    }
    else
        return false;
}
```

### Search
```c++
LinkedList* searchElementChaining(SinglyLinkedList* hashTable[], const int& key, const unsigned& sizeOfHashTable) {
    // prevent the client from searching for the default value
    if (key == 0)
        return nullptr;

    return hashTable[getHashValue(key, sizeOfHashTable)]->get(key);
}
```
```c++
Element* searchElementLinearProbing(Element hashTable[], const int& key, const unsigned& sizeOfHashTable) {
    // prevent the client from searching for the default value
    if (key == 0)
        return nullptr;
    
    unsigned hashValue = getHashValue(key, sizeOfHashTable);
    while (hashTable[hashValue].key != key && hashValue < sizeOfHashTable)
        hashValue++;

    if (hashValue < sizeOfHashTable)
        return &(hashTable[hashValue]);
    else
        return nullptr;
}
```

### Removal
```c++
bool removeElementChaining(SinglyLinkedList* hashTable[], const int& key, const unsigned& sizeOfHashTable) {
    unsigned hashValue = getHashValue(key, sizeOfHashTable);
    auto searchResult = hashTable[hashValue]->get(key);

    if (searchResult == nullptr)
        return false;
    else {
        // prevent the client from removing default value
        if (key == 0)
            return false;

        bool isRemoved = false;
        hashTable[hashValue]->remove(key, isRemoved);
        if (hashTable[hashValue] == nullptr)
            hashTable[hashValue] = new SinglyLinkedList(Element());
        return isRemoved;
    }
}
```
```c++
bool removeElementLinearProbing(Element hashTable[], const int& key, const unsigned& sizeOfHashTable) {
    Element defaultElement;
    unsigned hashValue = getHashValue(key, sizeOfHashTable);
    while (hashTable[hashValue].key != key && hashValue < sizeOfHashTable)
        hashValue++;

    if (hashValue < sizeOfHashTable) {
        hashTable[hashValue] = defaultElement;

        return true;
    }
    else
        return false;
}
```

### Initialization of Hash Tables
```c++
void initializedHashTableChaining(SinglyLinkedList* hashTable[], const int& sizeOfHashTable) {
    int currentKey = 7;
    for (unsigned currentCount = 1; currentCount < 40; ++currentCount, currentKey = 7 * currentCount)
        insertElementChaining(hashTable, Element{ currentKey }, sizeOfHashTable);
}
```
```c++
void initializedHashTableLinearProbing(Element hashTable[], const int& sizeOfHashTable) {
    int currentKey = 7;
    for (unsigned currentCount = 1; currentCount < 40; ++currentCount, currentKey = 7 * currentCount)
        insertElementLinearProbing(hashTable, Element{ currentKey }, sizeOfHashTable);
}
```

### Result Printer
```c++
void printSearchResultChaining(SinglyLinkedList* hashTable[], const int &key, const unsigned &sizeOfHashTable) {
    auto searchResult = searchElementChaining(hashTable, key, sizeOfHashTable);

    std::cout << "search for " << key << " : ";
    if (searchResult == nullptr)
        std::cout << "failed" << std::endl;
    else {
        std::cout << "succeeded(address : " << searchResult << ")" << std::endl;
        std::cout << "key of the resulting element : ";
        searchResult->printInfo();
        std::cout << std::endl;
    }
}
```
```c++
void printSearchResultLinearProbing(Element hashTable[], const int &key, const unsigned &sizeOfHashTable) {
    auto searchResult = searchElementLinearProbing(hashTable, key, sizeOfHashTable);

    std::cout << "search for " << key << " : ";
    if (searchResult == nullptr)
        std::cout << "failed" << std::endl;
    else {
        std::cout << "succeeded(address : " << searchResult << ")" << std::endl;
        std::cout << "key of the resulting element : " << searchResult->key << std::endl;
    }
}
```

### Client
```c++
constexpr int sizeOfHashTable = 30; // this can be any value
int keyToSearch = 7;

SinglyLinkedList* hashTableChaining[sizeOfHashTable];
for (unsigned currentBucket = 0; currentBucket < sizeOfHashTable; currentBucket++)
    hashTableChaining[currentBucket] = new SinglyLinkedList(Element());

initializedHashTableChaining(hashTableChaining, sizeOfHashTable);
std::cout << "Chaining" << std::endl;
printSearchResultChaining(hashTableChaining, keyToSearch, sizeOfHashTable);
removeElementChaining(hashTableChaining, keyToSearch, sizeOfHashTable);
printSearchResultChaining(hashTableChaining, keyToSearch, sizeOfHashTable);

for (unsigned currentBucket = 0; currentBucket < sizeOfHashTable; currentBucket++)
    delete hashTableChaining[currentBucket];


std::cout << "\n\n" << std::endl;


Element hashTableLinearProbing[sizeOfHashTable] = { Element() }; // every slot is initialized with the default element

initializedHashTableLinearProbing(hashTableLinearProbing, sizeOfHashTable);
std::cout << "Linear Probing" << std::endl;
printSearchResultLinearProbing(hashTableLinearProbing, keyToSearch, sizeOfHashTable);
removeElementLinearProbing(hashTableLinearProbing, keyToSearch, sizeOfHashTable);
printSearchResultLinearProbing(hashTableLinearProbing, keyToSearch, sizeOfHashTable);


/*
print result
Chaining
search for 7 : succeeded(address : 015B1AA8)
key of the resulting element : 7
search for 7 : failed



Linear Probing
search for 7 : succeeded(address : 0137FC84)
key of the resulting element : 7
search for 7 : failed
*/
```

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}