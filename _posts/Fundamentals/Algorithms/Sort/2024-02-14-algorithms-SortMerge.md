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
	void sortInsertion();
// same definition
};
```

### Merge Sort
```c++

```

### Client
```c++

```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}