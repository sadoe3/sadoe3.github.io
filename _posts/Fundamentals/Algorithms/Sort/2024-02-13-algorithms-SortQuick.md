---
title: "Algorithms : Quick Sort"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Sort, Quick Sort]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-13
---

# Quick Sort

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
The algorithm for the **quick sort** requires 2 functions
- `partition()` function
    * which is the core function of this algorithm
- `quickSort()` function
    * which uses recursion

### `partition()` Function
This function takes the collection, first index, and last index as parameters
1. it sets the current `pivot` index based on the given first and last index
    * `pivot = (firstIndex + lastIndex) / 2`
2. it sets the `leftIndex` and `rightIndex` based on the given first and last index
    * `leftIndex = firstIndex`
    * `rightIndex = lastIndex`
3. it performs iterations until this condition : `leftIndex < rightIndex` is `true`
    * during iterations
    * find the proper `leftIndex` to swap
        + the condition is : `collection[leftIndex] > collection[pivot]`
        + if that condition is `true`, then the current `leftIndex` is the one to swap
    * find the proper `rightIndex` to swap
        + the condition is : `collection[rightIndex] < collection[pivot]`
        + if that condition is `true`, then the current `rightIndex` is the one to swap
    * if `leftIndex < rightIndex` is `ture`
        + then, swap `collection[leftIndex]` and `collection[rightIndex]`
4. after the iteration ends, `leftIndex` would be same as `rightIndex`
    * then, there are 2 possible results
        1. `rightIndex > pivot`
            + in this case, if `collection[rightIndex] < collection[pivot]`
                - then, swap `collection[rightIndex]` with `collection[pivot]`
                - and, `return rightIndex`
            + otherwise
                - swap `collection[rightIndex - 1]` with `collection[pivot]`
                - and, `return (rightIndex - 1)`
        2. `rightIndex <= pivot`
            + in this case, if  `collection[rightIndex] > collection[pivot]`
                - then, swap `collection[rightIndex]` with `collection[pivot]`
                - and, `return rightIndex`
            + otherwise
                - swap `collection[rightIndex + 1]` with `collection[pivot]`
                - and, `return (rightIndex + 1)`

### `quickSort()` Function
This function takes the collection, first index, and last index as parameters too
1. it checks whether `firstIndex < lastIndex` or not
    + if that condition is `false`, then the recursion is done
2. othwerwise, this function gets new `pivot` by calling `partition()` function with the given parameters
3. then, it calls itself with the same collection but different indices twice
    + `quickSort(elements, firstIndex, (pivot-1))`
    + `quickSort(elements, (pivot+1), lastIndex)`


## Time Complexity
`O(nlogn)`


## Implementation

### Vector
```c++
template <typename Type>
class Vector {
public:
	void doQuickSort();
// same definition
};
```
```c++
/*
definitions of partition() and sortQuick()
*/
template <typename Type>
void Vector<Type>::doQuickSort() {
    sortQuick(elements, 0, (count - 1));
}
```

### Partition
```c++
template <typename Type>
unsigned partition(Type elements[], const unsigned &firstIndex, const unsigned &lastIndex) {
    unsigned pivot = (firstIndex + lastIndex) / 2;
    unsigned leftIndex = firstIndex, rightIndex = lastIndex;
    Type cachedElement;

    while (leftIndex < rightIndex) {
        while (elements[leftIndex] <= elements[pivot] && leftIndex < rightIndex)
            leftIndex++;

        while (elements[rightIndex] >= elements[pivot] && leftIndex < rightIndex)
            rightIndex--;

        if (leftIndex < rightIndex) {
            cachedElement = elements[leftIndex];
            elements[leftIndex] = elements[rightIndex];
            elements[rightIndex] = cachedElement;
        }
    }

    if (rightIndex > pivot) {
        if (elements[rightIndex] < elements[pivot]) {
            cachedElement = elements[pivot];
            elements[pivot] = elements[rightIndex];
            elements[rightIndex] = cachedElement;
            
            return rightIndex;
        }
        else {
            cachedElement = elements[pivot];
            elements[pivot] = elements[rightIndex - 1];
            elements[rightIndex - 1] = cachedElement;

            return (rightIndex - 1);
        }
    }
    else {
        if (elements[rightIndex] > elements[pivot]) {
            cachedElement = elements[pivot];
            elements[pivot] = elements[rightIndex];
            elements[rightIndex] = cachedElement;

            return rightIndex;
        }
        else {
            cachedElement = elements[pivot];
            elements[pivot] = elements[rightIndex + 1];
            elements[rightIndex + 1] = cachedElement;

            return (rightIndex + 1);
        }
    }
}
```

### Quick Sort
```c++
template <typename Type>
void sortQuick(Type elements[], const unsigned &firstIndex, const unsigned &lastIndex) {
    if (firstIndex < lastIndex) {
        unsigned pivot = partition(elements, firstIndex, lastIndex);
        sortQuick(elements, firstIndex, (pivot - 1));
        sortQuick(elements, (pivot + 1), lastIndex);
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

collection.doQuickSort();
printCollection(collection);

/*
print result
1 3 -1 -5 10 7 -10
-10 -5 -1 1 3 7 10
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}