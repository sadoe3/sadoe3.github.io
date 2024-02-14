---
title: "Algorithms : Insertion Sort"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Sort, Insertion Sort]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-14
---

# Insertion Sort

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
**Conceptually**, the algoritm for the **insertion sort** uses a collection of elements which consists of **two subsets**
- **sorted** subset
    * which is initialized with the first element
- **unsorted** subset
    * which is initialized with the other elements

### Iteration
This algorithm requires the **iteration** to be done
1. select the first element in the **unsorted** subset
2. **insert** it into the **sorted** subset in a proper way
    * you can choose the order of sort in this step
3. repeat this iteration until the **number** of elements in the **unsorted** subset becomes `0`


## Time Complexity
`O(n^2)`


## Implementation
There are 2 prerequisites for the implementation of this algorithm in view of the performance
- there's **no actual insertion**
    * there is only swapping
- **two** subsets does **not exist**
    * there is only one collection of elements 

### Vector
```c++
```c++
template <typename Type>
class Vector {
public:
	void sortInsertion();
// same definition
};
```

### Insertion Sort
```c++
template <typename Type>
void Vector<Type>::sortInsertion() {
    Type cachedElement;

    for (unsigned indexUnsorted = 1, indexSorted; indexUnsorted < count; indexUnsorted++) {
        cachedElement = elements[indexUnsorted];
        indexSorted = indexUnsorted;

        while ((indexSorted > 0) && (elements[indexSorted - 1] >= cachedElement)) {
            elements[indexSorted] = elements[indexSorted - 1];
            indexSorted--;
        }
        elements[indexSorted] = cachedElement;  // conceptual insertion
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

collection.sortInsertion();
printCollection(collection);

/*
print result
1 3 -1 -5 10 7 -10
-10 -5 -1 1 3 7 10
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}