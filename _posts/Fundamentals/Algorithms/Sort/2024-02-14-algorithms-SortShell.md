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
1. 


## Time Complexity
- Average: `O(n^1.5)`
- Worst: `O(n^2)`

## Implementation

### Vector
```c++
template <typename Type>
class Vector {
public:
	void sortInsertion();
// same definition
};
```

### Shell Sort
```c++

```

### Client
```c++

```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}