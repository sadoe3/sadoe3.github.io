---
title: "Algorithms : Radix Sort"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Sort, Radix Sort]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-14
---

# Radix Sort

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept

### Radix Sort
The algorithm for the **radix sort** follows the steps below
1. create **buckets** properly based on the base 
    * if current system uses `base 10`
        + then, create `10` buckets
2. set the `currentDigit` as `1`, then perform the iteration until `currentDigit` becomes the **number of digits** of **maximum value** from the collection
    1. store the elements of the collection into the **buckets** properly based on the `currentDigit`
    2. store the elements of the buckets into the **collection** in a proper way
        + you can choose the order of sort in this step
    3. **clear** all elements from all **buckets**
    4. increment `currentDigit`
3. done


## Time Complexity
`O(d*n)`
- where `d` is the number of digits of maximum value from the collection
- and `n` is the number of elements


## Implementation
This post uses `std::string` to get the `n`th digit from the value
- `1`st digit = one's place
- moreover, this post implemented a radix sort which handles **integral** types only

### Vector
```c++
template <typename Type>
class Vector {
public:
	void sortInsertion();
// same definition
};
```

### Maximum Digit
```c++
#include <cmath>
// some codes
unsigned getMaximumNumberOfDigit(Type elements[], const int& numberOfElements, const int& base) {

```

### Radix Sort
```c++
#include <string>
// some codes

```

### Client
```c++

```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}