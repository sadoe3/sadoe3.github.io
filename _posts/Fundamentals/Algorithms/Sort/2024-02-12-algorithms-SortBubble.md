---
title: "Algorithms : Bubble Sort"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Sort, Bubble Sort]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-12
---

# Bubble Sort

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
**Bubble sort** requires `N-1` iterations where `N` is the number of elements
- `1`st iteration
	* compare elements from the first one to the `N`th one
	* first of all, compare the 1st one and the 2nd one
	    + if the relationship is not valid, then swap them
	    + if you want the ascending order, the proper relationship would be `1st one <= 2nd one`
	* then, compare the 2nd one and the 3rd one
    * keep doing this until comparing the (`N-1`)th one and `N`th one 
- `2`nd iteration
	* compare elements from the first one to the (`N-1`)th one
	* Keep doing this until comparing the (`N-2`)th one and (`N-1`)th one
- ...
- `(N-1)`th iteration
    * compare the first element with the second one
	    + swap them if needed
- done


## Time Complexity
`O(n^2)`


## Implementation

### Vector
```c++
template <typename Type>
class Vector {
public:
	void sortBubble();
// same definition
};
```

### sortBubble()
```c++
template <typename Type>
void Vector<Type>::sortBubble() {
    if(count < 2)
        return;

    const unsigned END_POSITION = count - 1;
    Type cachedElement;
    for(unsigned iterationCount = 0; iterationCount < END_POSITION; iterationCount++) {
        for(unsigned currentIndex = 0; currentIndex < END_POSITION - iterationCount; currentIndex++) {
            if(elements[currentIndex] > elements[currentIndex+1]) {
                cachedElement = elements[currentIndex+1];
                elements[currentIndex+1] = elements[currentIndex];
                elements[currentIndex] = cachedElement;
            }
        }
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

collection.sortBubble();
printCollection(collection);

/*
print result
1 3 -1 -5 10 7 -10
-10 -5 -1 1 3 7 10
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}