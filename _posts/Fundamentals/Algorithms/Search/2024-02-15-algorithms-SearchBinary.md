---
title: "Algorithms : Binary Search"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Search, Binary Search]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-15
---

# Binary Search

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
It's worth noting that **binary search** is completely **different** from [**binary search tree**](https://sadoe3.github.io/data-structures/structures-BinarySearchTree/)
- the algorithm for the **binary search** also requires to **sort** the collection of the elements

### Steps
1. do the iteration if the condition `lowIndex != highIndex` is `true`
    * `lowIndex` is initialized with `0`
    * `highIndex` is initialized with `the number of elements - 1`
    1. compare the **key** with the key of the element at the `middleIndex`
        + `middleIndex = (lowIndex + highIndex) / 2`
        1. if they are equal, then
            - return the `middleIndex`
        2. otherwise,
            - modify the `lowIndex` or `highIndex` based on the result of the comparison 
2. compare the **key** with the key of the element at the `lowIndex`
    * if the control flow reads until this process, `lowIndex` and `highIndex` must be equal to each other
    1. if they are equal, then
        + return `lowIndex`
    2. otherwise
        + return `-1`
3. done


## Time Complexity
`O(logn)`


## Implementation

### Vector
```c++
template <typename Type>
class Vector {
public:
    int searchBinary(const Type &);
// same definition
};
```

### Binary Search
```c++
template <typename Type>
int Vector<Type>::searchBinary(const Type& targetValue) {
    if (count < 2) {
        if (count == 0)
            return -1;
        else
            return elements[0] == targetValue ? 0 : -1;
    }

    sortSelection();
    
    unsigned lowIndex = 0, highIndex = (count - 1), middleIndex;
    while (lowIndex != highIndex) {
        middleIndex = (lowIndex + highIndex) / 2;
        
        if (elements[middleIndex] == targetValue)
            return middleIndex;

        if (elements[middleIndex] < targetValue)
            lowIndex = middleIndex + 1;
        else
            highIndex = middleIndex - 1;
    }

    if (lowIndex == targetValue)
        return lowIndex;
    else
        return -1;
}
```

### Search Result Printer
```c++
template <typename Type>
void printSearchResult(Vector<Type> &collection, const Type& key) {
    auto searchKey = key;
    auto searchResult = collection.searchBinary(searchKey);

    std::cout << "search for " << searchKey << " : ";
    if (searchResult == -1)
        std::cout << "failed" << std::endl;
    else {
        std::cout << "succeeded(index : " << searchResult << ")" << std::endl;
        std::cout << "value at the index of " << searchResult << " : " << collection.get(searchResult) << std::endl;
    }
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

printSearchResult(collection, 2);
printSearchResult(collection, 7);


/*
print result
1 3 -1 -5 10 7 -10
search for 2 : failed
search for 7 : succeeded(index : 5)
value at the index of 5 : 7
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}