---
title: "Algorithms : Indexed Linear Search"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Search, Indexed Linear Search]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-15
---

# Indexed Linear Search

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
The algorithm for the **indexed linear search** is based on the [**linear search**](https://sadoe3.github.io/algorithms/algorithms-SearchLinear/)
- the difference is that
    * indexed linear search uses **index table** to search for the target

### Index Table
The prerequisite of the **index table** is to **sort** the collection of the elements
- the **index table** stores the pair of the key and the index of some elements from the collection with a certain **inverval**
- with the index table, we can do the linear search within a **smaller** range which may contain the key than the original range

### Steps
1. sort the collection of the elements
2. construct the index table
3. find the smaller range by using the index table
4. do the linear search within that smaller range
5. done


## Time Complexity
`O(t+i)`
- where `t` is the number of elements in the **index table**
- and `i` is the value of the **interval** for the index table


## Implementation

### Vector
```c++
template <typename Type>
class Vector {
public:
    int searchIndexedLinear(const Type &, const unsigned &);
// same definition
};
```

### Index Table Data
```c++
// this type assumes that the type for the type can be default initialized with 0
template <typename Type>
struct IndexTableData {
public:
    IndexTableData(const Type &inputKey = 0, const unsigned& inputIndex = 0) : key(inputKey), index(inputIndex) { }
    Type key;
    unsigned index;
};
```

### Index Table Creator
```c++
template <typename Type>
unsigned constructIndexTable(Vector<IndexTableData<Type>>& indexTable, Type collection[], const unsigned &numberOfElements, const unsigned &interval) {
    indexTable.reset();

    unsigned count = 0;
    for (unsigned currentIndex = 0; currentIndex < numberOfElements; currentIndex += (interval + 1)) {
        indexTable.pushBack(IndexTableData<Type>(collection[currentIndex], currentIndex));
        count++;
    }
    return count;
}
```

### Indexed Linear Search
```c++
template <typename Type>
int Vector<Type>::searchIndexedLinear(const Type& targetValue, const unsigned &interval) {
    sortSelection();
    
    Vector<IndexTableData<Type>> indexTable;
    const unsigned COUNT_INDEX_TABLE = constructIndexTable(indexTable, elements, count, interval);
    int currentIndexForIndexTable = 0;
    for (; currentIndexForIndexTable < COUNT_INDEX_TABLE; currentIndexForIndexTable++) {
        if (indexTable.get(currentIndexForIndexTable).key > targetValue)
            break;
    }
    currentIndexForIndexTable--;

    if (currentIndexForIndexTable == -1)
        return -1;

    // do the linear search within a smaller range
    unsigned currentIndex = indexTable.get(currentIndexForIndexTable).index;
    const unsigned END_INDEX = currentIndex + interval + 1;
    for (; currentIndex < END_INDEX; currentIndex++) {
        if (elements[currentIndex] == targetValue)
            break;
    }

    return currentIndex < END_INDEX ? currentIndex : -1;
}
```

### Search Result Printer
```c++
template <typename Type>
void printSearchResult(Vector<Type> &collection, const Type& key, const unsigned &interval) {
    auto searchKey = key;
    auto searchResult = collection.searchIndexedLinear(searchKey, interval);
    std::cout << "search for " << searchKey << " : ";
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

printSearchResult(collection, 2, 3);
printSearchResult(collection, 7, 3);


/*
print result
1 3 -1 -5 10 7 -10
search for 2 : failed
search for 7 : succeeded(index : 5)
value at the index of 5 : 7
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}