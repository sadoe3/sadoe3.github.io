---
title: "Algorithms : Heap Sort"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Sort, Heap Sort]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-19
---

# Heap Sort

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
The algorithm for the **heap sort** is based on the [**heap**](https://sadoe3.github.io/data-structures/structures-Heap/) data structure

### Order
- if you decide to use **max heap**
    * then you can sort the collection in a **descending** order
- if you decide to use **min heap**
    * then you can sort the collection in an **ascending** order


## Time Complexity
`O(nlogn)`


## Implementation

### Heap
[**Heap**](https://sadoe3.github.io/data-structures/structures-Heap/#implementation)

### Vector
```c++
template <typename Type>
class Vector {
public:
	void sortInsertion();
// same definition
};
```

### Heap Sort
```c++
template <typename Type>
void Vector<Type>::sortHeap() {
    MinHeap<Type> heap;

    for (unsigned currentIndex = 0; currentIndex < count; currentIndex++)
        heap.add(elements[currentIndex]);

    for (unsigned currentIndex = 0; currentIndex < count; currentIndex++) {
        elements[currentIndex] = *(heap.getRootNode());
        heap.remove();
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

collection.sortHeap();
printCollection(collection);

/*
print result
1 3 -1 -5 10 7 -10
-10 -5 -1 1 3 7 10
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}