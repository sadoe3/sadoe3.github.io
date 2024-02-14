---
title: "Algorithms : Selection Sort"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Sort, Selection Sort]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-11
---

# Selection Sort

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
The algorithm for the **selection sort** is :
1. set the first element as the current target position
2. compare the element at the target position with the elements after it
    * if you want the ascending order, make the first element the smallest one
	    + otherwise, make it the largest one
	* if the result of the comparison is invalid, swap them
	    + otherwise, continue to keep comparing
3. after the comparisons, set the second element as the current target position
	* do the something as we do for the first element
4. keep iterating this process until the target position is the element prior to the last element
5. do the final comparison, and swap if needed
6. done


## Time Complexity
`O(n^2)`


## Implementation

### Vector
```c++
template <typename Type>
class Vector {
public:
    Vector() : elements(nullptr), size(4), count(0) { 
        elements = new Type[size];
    }
    ~Vector() {
        delete[] elements;
    }

    void pushBack(const Type& newElement);

    Type popBack() {
        if(count > 0) {
            count--;
            return elements[count];
        }
        throw std::exception("trying to remove the element at the invalid position");
    }

    Type& get(unsigned index) {
        if(index < count)
            return elements[index];

        throw std::exception("trying to remove the element at the invalid position");
    }

    void sortSelection();
private:
    Type* elements;
    unsigned size;
    unsigned count;
};
template <typename Type>
void Vector<Type>::pushBack(const Type& newElement) {
    elements[count] = newElement;
    count++;

    if(count == size) {
        size *= 2;
        Type* newCollection = new Type[size];
        for(unsigned currentIndex = 0; currentIndex < count; currentIndex++)
            newCollection[currentIndex] = elements[currentIndex];
        delete[] elements;
        elements = newCollection;
    }
}
```

### Collection Printer
```c++
template <typename Type>
void printCollection(Vector<Type>& targetCollection) {
    unsigned currentIndex = 0;

    while(true) {
        try {
            std::cout << targetCollection.get(currentIndex) << " ";
        }
        catch(std::exception) {
            std::cout << std::endl;
            break;
        }
        currentIndex++;
    }
}
```

### Selection Sort
```c++
template <typename Type>
void Vector<Type>::sortSelection() {
    if(count < 2)
        return;

    unsigned targetPosition = 0, currentIndex;
    const unsigned END_POSITION = count - 1;
    Type cachedElement;

    while(targetPosition < END_POSITION) {
        for(currentIndex = targetPosition + 1; currentIndex <= END_POSITION; currentIndex++) {
            if(elements[targetPosition] > elements[currentIndex]) {
                cachedElement = elements[targetPosition];
                elements[targetPosition] = elements[currentIndex];
                elements[currentIndex] = cachedElement;
            }
        }
        targetPosition++;
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

collection.sortSelection();
printCollection(collection);

/*
print result
1 3 -1 -5 10 7 -10
-10 -5 -1 1 3 7 10
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}