---
title: "Algorithms : Merge Sort"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Sort, Merge Sort]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-14
---

# Merge Sort

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
The algorithm for the **merge sort** uses **multiple subsets**

### Merge Sort
The algorithm follows the steps below
1. **divide** the current collection of elements into **2 subsets**
    * repeat this process until the **number** of all subsets' element is `one`
2. **merge** two subsets into the one in a proper order
    * you can choose the order of sort in this step
    * repeat this process until the **all subsets** are merged into the **one set**
3. done


## Time Complexity
`O(nlogn)`


## Implementation
Given the performance of the algorithm, there's **no actual division** for the subsets
- `merge()` function merges two **conceptual** subsets into the one
    * this function may require the temporary collection to contain the sorted result
- this algorithm is **easily understandable** when we think of the concept of [**stack**](https://sadoe3.github.io/data-structures/structures-Stack/)

### Vector
```c++
template <typename Type>
class Vector {
public:
    void reset() {
        count = 0;
    }
    void doSortMerge();
// same definition
};
```

### Merge Function
```c++
template <typename Type>
void merge(Type elements[], Vector<Type> &temporaryCollection, const unsigned &firstIndex, const unsigned &middleIndex, const unsigned &lastIndex) {
    unsigned leftIndex = firstIndex, rightIndex = (middleIndex + 1);

    temporaryCollection.reset();
    unsigned countTemporaryCollection = 0;
    while ((leftIndex <= middleIndex) && (rightIndex <= lastIndex)) {
        if (elements[leftIndex] <= elements[rightIndex]) {
            temporaryCollection.pushBack(elements[leftIndex]);
            leftIndex++;
        }
        else {
            temporaryCollection.pushBack(elements[rightIndex]);
            rightIndex++;
        }
        countTemporaryCollection++;
    }
    if (leftIndex <= middleIndex) {
        for (; leftIndex <= middleIndex; leftIndex++) {
            temporaryCollection.pushBack(elements[leftIndex]);
            countTemporaryCollection++;
        }
    }
    else {
        for (; rightIndex <= lastIndex; rightIndex++) {
            temporaryCollection.pushBack(elements[rightIndex]);
            countTemporaryCollection++;
        }
    }

    for (unsigned currentIndex = firstIndex, currentCount = 0; currentCount < countTemporaryCollection; currentIndex++, currentCount++)
        elements[currentIndex] = temporaryCollection.get(currentCount);
}
```

### Merge Sort
```c++
template <typename Type>
void sortMerge(Type elements[], Vector<Type> &temporaryCollection, const unsigned &firstIndex, const unsigned &lastIndex) {
    if (firstIndex < lastIndex) {
        unsigned middleIndex = (firstIndex + lastIndex) / 2;

        sortMerge(elements, temporaryCollection, firstIndex, middleIndex);
        sortMerge(elements, temporaryCollection, middleIndex + 1, lastIndex);
        merge(elements, temporaryCollection, firstIndex, middleIndex, lastIndex);
    }
}
```

### Vector Again
```c++
template <typename Type>
void Vector<Type>::doSortMerge() {
    if (count > 1) {
        Vector<Type> temporaryCollection;
        sortMerge(elements, temporaryCollection, 0, count - 1);
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

collection.doSortMerge();
printCollection(collection);

/*
print result
1 3 -1 -5 10 7 -10
-10 -5 -1 1 3 7 10
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}