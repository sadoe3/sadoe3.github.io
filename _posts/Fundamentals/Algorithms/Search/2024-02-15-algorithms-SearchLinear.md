---
title: "Algorithms : Linear Search"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Search, Linear Search]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-15
---

# Linear Search

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
The algorithm for the **linear search** does **not require to sort** the collection before beginning the search

### Sequential Search
What it does is to **iterate the whole collection** until it finds the target element
- if it's found
    * then the function returns the **index** of the first element which has that target key
- otherwise
    * it returns `-1`


## Time Complexity
`O(n)`


## Implementation

### Vector
```c++
template <typename Type>
class Vector {
public:
    int searchLinear(const Type &);
// same definition
};
```

### Linear Search
```c++
template <typename Type>
int Vector<Type>::searchLinear(const Type &targetValue) {
    unsigned resultIndex = 0;
    for (; resultIndex < count; resultIndex++) {
        if (elements[resultIndex] == targetValue)
            break;
    }

    return resultIndex == count ? -1  : resultIndex;
}
```

### Search Result Printer
```c++
template <typename Type>
void printSearchResult(Vector<Type> &collection, const Type &key) {
    std::cout << "search for " << key << " : ";

    auto searchResult = collection.searchLinear(key);
    if (searchResult == -1)
        std::cout << "failed" << std::endl;
    else {
        std::cout << "succeeded(index : " << searchResult << ")" << std::endl;
        std::cout << "value at the index of " << searchResult << " : " << collection.get(searchResult) << std::endl;
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

printSearchResult(collection, 2);
printSearchResult(collection, 7);


/*
print result
1 3 -1 -5 10 7 -10
search for 2 : failed
search for 7 : succeeded(index : 5)
value at the index of 5 : 7
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}