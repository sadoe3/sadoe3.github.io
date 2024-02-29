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

date: 2024-02-17
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
    Vector& operator=(const Vector&);
    void sortRadix(const unsigned &base);
// same definition
};
```
```c++
template <typename Type>
Vector<Type>& Vector<Type>::operator=(const Vector<Type>& rhs) {
    size = rhs.size;
    count = rhs.count;

    delete[] elements;
    elements = new Type[size];
    for (unsigned currentCount = 0; currentCount < count; currentCount++)
        elements[currentCount] = rhs.elements[currentCount];

    return *this;
}
```

### Maximum Digit
```c++
#include <cmath>
// some codes

// this function assumes that element's type is integral
template <typename Type>
unsigned getMaximumNumberOfDigit(Type elements[], const unsigned& numberOfElements, const int& base) {
    unsigned maxIndex = 0;
    for (unsigned currentIndex = 0; currentIndex < numberOfElements; currentIndex++) {
        if (elements[maxIndex] < elements[currentIndex])
            maxIndex = currentIndex;
    }

    return (floor(log10(elements[maxIndex])) + 1);
}
```

### Radix Sort
```c++
#include <string>
// some codes

// this function handles the collection which contains negative values or (positive values or 0) only
template <typename Type>
void sortRadixHalfCollection(Type elements[], Vector<Vector<Type>>& buckets, const unsigned& numberOfElements, const unsigned &base) {
    unsigned maxDigit = getMaximumNumberOfDigit(elements, numberOfElements, base);
    Vector<unsigned> countBuckets;
    for (unsigned currentCount = 0; currentCount < base; currentCount++)
        countBuckets.pushBack(0);

    for (int currentDigit = 1, stringIndex; currentDigit <= maxDigit; currentDigit++) {
        for (unsigned currentIndex = 0, bucketIndex; currentIndex < numberOfElements; currentIndex++) {
            stringIndex = (std::to_string(elements[currentIndex]).size() - currentDigit);

            if (stringIndex < 0) {
                buckets.get(0).pushBack(elements[currentIndex]);
                countBuckets.get(0) += 1;
            }
            else {
                bucketIndex = (std::to_string(elements[currentIndex]).at(stringIndex) - '0');

                buckets.get(bucketIndex).pushBack(elements[currentIndex]);
                countBuckets.get(bucketIndex) += 1;
            }
        }

        for (unsigned currentBucketIndex = 0, currentCollectionIndex = 0, currentIndex, endIndex; currentCollectionIndex < numberOfElements; currentBucketIndex++) {
            Vector<Type>& currentBucket = buckets.get(currentBucketIndex);
            for (currentIndex = 0, endIndex = countBuckets.get(currentBucketIndex); currentIndex < endIndex; currentIndex++, currentCollectionIndex++)
                elements[currentCollectionIndex] = currentBucket.get(currentIndex);
        }

        for (unsigned currentBucketIndex = 0; currentBucketIndex < base; currentBucketIndex++) {
            buckets.get(currentBucketIndex).reset();
            countBuckets.get(currentBucketIndex) = 0;
        }
    }
}
```
```c++
template <typename Type>
void Vector<Type>::sortRadix(const unsigned& base) {
    if (count < 2)
        return;

    Vector<Vector<Type>> buckets;
    for (unsigned currentBase = 0; currentBase < base; currentBase++)
        buckets.pushBack(Vector<Type>());

 
    Vector<Type> collectionNegative, collectionPositive;
    unsigned countCollectionNegative = 0, countCollectionPositive = 0;
    for (unsigned currentIndex = 0; currentIndex < count; currentIndex++) {
        if (elements[currentIndex] < 0) {
            collectionNegative.pushBack(elements[currentIndex] * (-1));
            countCollectionNegative++;
        }
        else {
            collectionPositive.pushBack(elements[currentIndex]);
            countCollectionPositive++;
        }
    }

    sortRadixHalfCollection(collectionNegative.elements, buckets, countCollectionNegative, base);
    sortRadixHalfCollection(collectionPositive.elements, buckets, countCollectionPositive, base);

    unsigned currentIndex = 0;
    for (int currentIndexNegative = countCollectionNegative - 1; currentIndexNegative > -1; currentIndexNegative--, currentIndex++)
        elements[currentIndex] = (collectionNegative.get(currentIndexNegative) * (-1));
    for (unsigned currentIndexPositive = 0; currentIndexPositive < countCollectionPositive; currentIndexPositive++, currentIndex++)
        elements[currentIndex] = collectionPositive.get(currentIndexPositive);
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

collection.sortRadix(10);
printCollection(collection);

/*
print result
1 3 -1 -5 10 7 -10
-10 -5 -1 1 3 7 10
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}