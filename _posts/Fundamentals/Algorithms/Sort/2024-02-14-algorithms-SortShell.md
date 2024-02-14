---
title: "Algorithms : Shell Sort"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Sort, Shell Sort]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-14
---

# Shell Sort

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
The algorithm for the **shell sort** is based on the [**insertion sort**](https://sadoe3.github.io/algorithms/algorithms-SortInsertion/)
- the difference is that
    * shell sort utilizes the **interval** to move to the next element
- this algorithm uses 2 functions
    * `sortShell()`
    * `sortInterval()`

### `sortShell()` Function
This function sets the **interval** and calls `sortInterval()` based the calculated **interval**
1. `interval` = number of elements
2. do the iteration until `interval > 1`
    1. `interval /= 2`
    2. do the iteration (syntax: `for(unsigned initialIndexSorted = 0; initialIndexSorted < interval; initialIndexSorted++)`)
        1. call `sortInterval()` with the proper arguments 
3. done

### `sortInterval()` Function
This function takes the collection of elements, the initial index of the sorted subset, the number of elements, and the interval as the parameters
1. do the iteration unless the number of elements in unsorted subset is `0`
    1. select first element in unsorted subset by using the initial index of the sorted subset
    2. **insert** it into the sorted subset in a proper order
        + you can choose the order of sort in this step
    3. move to the next element with the size of the **interval**
2. done


## Time Complexity
- Average: `O(n^1.5)`
- Worst: `O(n^2)`


## Implementation
Because it's based on the insertion sort, there are 2 prerequisites for the implementation of this algorithm in view of the performance
- there's **no actual insertion**
    * there is only swapping
- **two** subsets does **not exist**
    * there is only one collection of elements 

### Vector
```c++
template <typename Type>
class Vector {
public:
    void doSortShell();
// same definition
};
```

### Interval Sort
```c++
template <typename Type>
void sortInterval(Type elements[], const int &initialIndexSorted, const int &numberOfElements, const int &interval) {
    Type cachedElement;

    for (int indexUnsorted = initialIndexSorted + interval, indexSorted; indexUnsorted < numberOfElements; indexUnsorted += interval) {
        cachedElement = elements[indexUnsorted];
        indexSorted = (indexUnsorted - interval);

        while ((indexSorted >= initialIndexSorted) && (elements[indexSorted] >= cachedElement)) {
            elements[indexSorted + interval] = elements[indexSorted];
            indexSorted -= interval;
        }
        elements[indexSorted + interval] = cachedElement;  // conceptual insertion
    }
}
```

### Shell Sort
```c++
template <typename Type>
void sortShell(Type elements[], const int &numberOfElements) {
    unsigned interval = numberOfElements;
    while (interval > 1) {
        interval /= 2;

        for (unsigned initialIndexSorted = 0; initialIndexSorted < interval; initialIndexSorted++)
            sortInterval(elements, initialIndexSorted, numberOfElements, interval);
    }
}

```

### Vector Again
```c++
template <typename Type>
void Vector<Type>::doSortShell() {
    sortShell(elements, count);
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

collection.doSortShell();
printCollection(collection);

/*
print result
1 3 -1 -5 10 7 -10
-10 -5 -1 1 3 7 10
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}