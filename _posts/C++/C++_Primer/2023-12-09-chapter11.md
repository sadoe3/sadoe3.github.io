---
title: "C++ Primer : Chapter 11"

categories:
    - cpp

tags:
    - [C++, Programming Language, Associative Container, map]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-9
---

# Associative Containers

> 이 포스트는 C++ Primer(5th Edition)를 바탕으로 작성되었습니다.

## Using an Associative Container

### Core Concept of Associative Containers
Associative Containers store the element based on the concept of **key-value** relationships
- sequential containers store the element based on the concept of **sequntially positioned**
- although the associative containers share much of the behavior of the sequential containers, they differ from the sequential containers in ways that reflect the use of keys

### Types of Associative Containers

||Associative Container Types|
|:---:|:---|
|Elements Ordered by Key||
|`map`|associative array; holds key-value paris|
|`set`|container in which the key is the value|
|`multimap`|`map` in which a key can appear multiple times|
|`multiset`|`set` in which a key can appear multiple times|
|Unordered Collections||
|`unordered_map`|`map` organized by a hash function|
|`unordered_set`|`set` organized by a hash function|
|`unordered_multimap`|hashed `map`; keys can appear multiple times|
|`unordered_multiset`|hashed `set`; keys can appear multiple times|

- `std::map` and `std::multimap` types are defined in the `<map>` header
- `std::set` and `std::multiset` types are defined in the `<set>` header
- the unordered containers are defined in the `<unordered_map>` and `<unordered_set>` header
- like the sequential container, the associative containers are **templates**
    * to define a `std::map`, you need to specify the types of the key and value
    ```c++
    // the first one the key, and the second one is the value
    std::map<std::string, unsigned> word_count;
    std::string word;
    while(std::cin >> word) 
        ++word_count[word];
    for(const auto & w : word_cound)
        std::cout << w.first << " occurs " << w.second << std::endl;
    ```
- when you fetch an element from a `std::map`, you get an object of type `std::pair` which would be covered soon
    * `std::pair` template type holds two data members named `.first` and `.second`
    * for `std::map`, `.first` is the key and `.second` is the value
- the iterators for associative container are bidirectional
- for ordered containers, the key type must define a way to **compare** the elements
- for unordered containers, the key type must define a way to perform **`==` operation** and there must be a way to create a **hash code** based on the key type 

### Construction of an Associative container
We can construct the associative container in various ways
- default construction
    * same as sequential container
- copy construction
    * same as sequential container
- list construction
    * you can construct the associative container with the braced list
    ```c++
    std::set<std::string> exclude = { "the", "but", "and" };
    std::map<std::string, std::string> authors = { {"Joyce", "James"}, {"Kyle", "Charles"} };   // similar to initialize multi-dimensional array
    ```
- construction with the predicate
    * you can override the `<` operator so that the ordering is based on the custom operation
    * you need to pass the type of the predicate as the last template argument
        ```c++
        // there are two ways to implement this
        
        // using pointer to function
        bool compIsbn(const BookData & lhs, const BookData & rhs) {
            return lhs.isbn() < rhs.isbn();
        }
        std::multiset<BookData, decltype(compIsbn) *> bookstore(comIsbn);

        // using comparison struct
        struct CompIsbn {
            bool operator()(const Point& lhs, const Point& rhs) const {
                return lhs.x < rhs.x;
            }
        };
        std::multiset<BookData, CompIsbn> bookstore;
        ```
    * the custom operation must define a **strict weak ordering** over the key type
        + two keys cannot both be "less than" each other; if `k1` is "less than" `k2`, then `k2` must never be "less than" `k1`
        + if `k1` is "less than" `k2` an `k2` is "less than `k3`, then `k1` must be "less than" `k3`
        + if there are two keys, and neither key is "less than" the other, then we'll say that those keys are "equivalent". if `k1` is "equivalent" to `k2` and `k2` is "equivalent" to `k3`, then `k1` must be "equivalent" to `k3`
    * in practice, what's important is that a type that defines a `<` operator that **behaves normally** can be used as a key

### The pair Type
`std::pair` is defined in the `<utility>` header

||Operations on `std::pair`s|
|:---:|:---|
|`pair<T1, T2> p;`|`p` is a `pair` with value initialized members of types `T1` and `T2`, respectively.|
|`pair<T1, T2> p(v1, v2);`|`p` is a pair with types `T1` and `T2`; the first and second members are initialized from `v1` and `v2`, respectively.|
|`pair<T1, T2> p = {v1, v2};`|equivalent to `p(v1, v2)`.|
|`make_pair(v1, v2)`|returns a `pair` initialized from `v1` and `v2`. The type of the `pair` is inferred from the types of `v1` and `v2`.|
|`p.first`|returns the (`public`) data member of `p` named first.|
|`p.second`|returns the (`public`) data member of `p` named second.|
|`p1 relop p2`|relational operators (`<, >, <=, >=`). relational operators are defined as dictionary ordering: for example, `p1 < p2` is `true` if `pl.first < p2.first` or if `!(p2.first < p1.first) && p1.second < p2.second`. Uses the element's `< operator`.|
|`p1 == p2`|two `pair`s are equal if their `first` and `second` members are|
|`p1 != p2`|respectively equal. uses the element's `== operator`.|


## Operations on Associative Containers

### Type Aliases

||Associative Container Additional Type Aliases|
|:---:|:---|
|`::key_type`|type of the key for this container type|
|`::value_type`|for `std::set`s, same as the `::key_type`<br>for `std::map`s, `std::pair<const key_type, mapped_type>`|
|`::mapped_type`|type associated with each key; **`std::map` types only**|

### Associative Container Iterators
When we dereference an iterator from the associative container, we get a reference to a value of the container's `::value_type`
- hence, for `std::map`s, we can change the value but not the key through the iterators
- although the `std::set` types define both the `::iterator` and `::const_iterator` types
    + both types of iterators give us **read-only** access to the elements
    + because the elements of the `std::set` are the `key`s which must be `const` 
- in general, we do not use the generic algorithms with the associative containers
    * because we cannot write to or reorder elements due to the fact that `key`s are `const`
    * also, algorithms which need only read, like `std::find`, usually don't used
        + because we can easily find the element by accessing through the `key` 

### Adding Elements

||Operations to Add Elements to the Associative Container|
|:---:|:---|
|`c.insert(v)`|`v` is the `::value_type` object of the container|
|`c.emplace(args)`|`args` are used to construct an element|
|`c.insert(b, e)`|`b` and `e` are iterators that denote a range of `c::value_type` values; returns `void`|
|`c.insert(li)`|`li` is a braced list of `c::value_type` objects; returns `void`|
|`c.insert(p, v)`|like `insert(v)`, but uses iterator `p` as a hint for where to begin the search for where the new element should be stored. returns an iterator to the element with the given key|
|`c.emplace(p, args)`|`.emplace()` version of `c.insert(p,v)`|
|`c[k]`|returns the element with key `k`; if `k` is not int `c`, adds a new value-initialized element with key `k`|
|`c[k] = t`|returns the element with key `k`; if `k` is not int `c`, adds an element initialized by `t` with key `k`|

- for containers with the unique key, `.insert()` and `.emplace()` methods return a **`std::pair` object** 
    * which contains the iterator which denotes to the inserted element as its `.first` and the `bool` object which is `true` if the insertion is complete as its `.second`
- for the multiple keys, they return the **iterator** which denotes to the new inserted element

### Erasing Elements
We can remove elements of the associative containers by calling their 3 different `.erase()` methods

||Operations to Remove Elements from the Associative Container|
|:---:|:---|
|`c.erase(k)`|removes every element with key `k` from `c`. returns `size_type` indicating the number of elements removed|
|`c.erase(p)`|removes the element denoted by the iterator `p` fomr `c`. `p` must refer to an actual element in `c`; it must not be equal to `c.end()`. returns an iterator to the element after `p` or `c.end()` if `p` denotes the last element of `c`|
|`c.erase(b, e)`|removes the elements in the range denoted by the iterator pair `b`, `e,`. returns `e`|

### Accessing Elements
The associative containers provide various ways to find a given element

||Operations to Access to Elements from the Associative Container|
|:---:|:---|
|`c[k]`|returns the element with key `k`; if `k` is not int `c`, adds a new value-initialized element with key `k`. **Valid for `std::map` and `std::unordered_map` that are not `const`**|
|`c.at(k)`|returns the element with key `k`; if `k` is not in `c`, throws an `out_of_range` exception. **Valid for `std::map` and `std::unordered_map` that are not `const`**|
|`c.find(k)`|returns an iterator to the (first) element with key `k`, or `c.end()` if `k` is not in the container|
|`c.count(k)`|returns the number of elements with key `k`|
|`c.lower_bound(k)`|returns an iterator to the first element with key not less than `k`. **Not valid for the unordered containers**|
|`c.upper_bound(k)`|returns an iterator to the first element with key not greater than `k`. **Not valid for the unordered containers**|
|`c.equal_range(k)`|returns a `std::pair` of iterators of which `.first` refers to the first instance of the key and `.second` refers to one past the last instance of the key. if `k` is not present, both members are `c.end()`|

- for `.lower_bound()` and `.upper_bound()`, if there is no element for the key, the iteratators returned from them will be **equal**
    * because they will refer to the point at which the given key can be inserted while maintaining the container order

## The Unordered Containers
The **unordered associative** containers use a **hash function** and the key type's `==` operator rather than using a comparsion operation

### When to Use Unordered Containers
It's usually easier (and often yields a better performance) to use an ordered container
- however, try to use the **unordered associative** containers if
    * there is no obvious ordering relationship among the elements or
    * performance is increaseed when **hashing** is utilized

### Operations on Unordered Associative Containers
Aside from operations that manage the **hashing**, the unordered containers provide the **same** operations, such as `.find()` or `.insert()`, as the ordered containers
- the unordered containers are organized as a collection of **buckets**, each of which holds zero or more elements
    * they use a **hash function** to map elements to **buckets** by computing the **hash code** of the element to find the proper bucket for the element
    * if the multiple keys are allowed, then all the elements with the same key will be in the same bucket
    * therefore, the performance of an unordered container depends on the quality of its **hash function** and on the number and size of its **buckets**
- the **hash function** must always yield the same result when called with the same argument
    * ideally, the hash function maps each particular element to a unique bucket
    * however, it's allowed to map elements with differing keys to the same bucket
    * when a buck holds several elements, those elements are searched sequentially to find the one we want
- the following operations are related to management of **hashing**

    ||Unordered Container Management Operations|
    |:---:|:---|
    |**Bucket Interface**||
    |`c.bucket_count()`|returns the number of buckets in use|
    |`c.max_bucket_count()`|returns largest number of buckets this container can hold|
    |`c.bucket_size(n)`|returns number of elements in the `n`th bucket|
    |`c.bucket(k)`|returns bucket in which elements with key `k` would be found|
    |**Bucket Iteration**||
    |`local_iterator`|iterator type that can access elements in a bucket|
    |`const_local_iterator`|`const` version of the bucket iterator|
    |`c.begin(n), c.end(n)`|iterator to the first, one past the last element in bucket `n`|
    |`c.cbegin(n), c.cend(n)`|returns `const_local_iterator`|
    |**Hash Policy**||
    |`c.load_factor()`|returns average number of elements per bucket. the resulting type is `float`.|
    |`c.max_load_factor()`|returns average bucket size that `c` tries to maintain. `c` adds buckets to keep `load_factor <= max_load_factor`. the resulting type is `float`.|
    |`c.rehash(n)`|reorganizes storage so that `bucket_count >= n` and and `bucket_count > size/max_load_factor`.|
    |`c.reserve(n)`|reorganize so that `c` can hold `n` elements without a `rehash`.|

### Customizing Hash Function and Equality Operator
Like the ordered container, it's possible to override the `==` operator and **hash function**
```c++
std::size_t hasher(const Book & book) {
    return std::hash<std::string>() (book.isbn());
}
bool eqOp(const Book & lhs, const Book & rhs) {
    return lhs.isbn() == rhs.isbn();
}
... // some codes
std::unordered_multiset<Book, decltype(hasher)*, decltype(eqOp) *> bookstore(42, hasher, eqOp); // arguments are the bucket size, pointers to the hash function and equality operator
std::unordered_multiset<Book, decltype(hasher)*> bookstore(42, hasher); // Book must have an == operator in this case
```
- by default, an object of type `std::hash<key_type>` is used to generate the hash code for each element

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}