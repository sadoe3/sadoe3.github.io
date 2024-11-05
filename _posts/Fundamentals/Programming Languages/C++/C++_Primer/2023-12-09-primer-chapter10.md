---
title: "C++ Primer : Chapter 10"

categories:
    - cpp

tags:
    - [C++, Programming Language, Algorithms, sort, search, find]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-9
---

# Generic Algorithms

> 이 포스트는 C++ Primer(5th Edition)를 바탕으로 작성되었습니다.

## Overview

### Generic Algorithms
The standard library defines a set of **generic algorithms**.
- **algorithms** becuase they implement common classical algorithms such as sorting and searching
- **generic** because they operate on elements of **various** types - not only library types as `std::vector` or `std::list`, but also the built-in array type, and other kinds of sequences as well
    * **iterators** make the algorithms container-independent

### How find() works
Most of the algorithms are defined in the `<algorithm>` header, also a set of generic numeric algorithms are defined in the `<numeric>` header
- in general, most algorithms operate by traversing a range of elements bounded by two **iterators**
    ```c++
    #include <algorithm>
    ...     // some codes which defines vec a vector object and ia an array
    int val = 42;
    // vector
    auto result = std::find(vec.cbegin(), vec.cend(), val);
    std::cout << "the value " << val << (result==vec.cend()) ? " is not present" : " is present" << std::endl;

    // array
    int * result = std::find(std::begin(ia), std::end(ia), val);    // find whole elements
    // or
    auto result = std::find(ia + 1, ia + 4, val);   // find val from from ia[1] to ia[3]
    std::cout << "the value " << val << (result==std::end(ia)) ? " is not present" : " is present" << std::endl;
    ```
- the above codes show the typical way to use `find()` method
    * if there is no match, `find()` returned its second iterator 

### How the Algorithms Work
- the algorithms take **two iterators** to get the range to traverse the given collection
- they use the operations, such as `==` or `<`, on the **element type** of the collection to perform their job
    * most of them provide a way for us to supply our **own operation** to use in place of the default operator 


## A first Look at the Algorithms
The library provides **more than 100** algorithms.
- fortunately, they have a **consistent** archiecture
- understanding this architecture makes learning and useing the algorithms easer than memorizing all 100+ of them
    - [this website](https://en.cppreference.com/w/cpp/algorithm) or Appendix A(p.866) describes the list of them

### Read-Only Algorithms
Some algorithms read, but never write to, the elements in their input range
- generally, it's best to use `cbegin()` and `cend()` with **read-only** algorithms because you can use the compiler to check whether it tries changing the value or not
- `std::accumulate()` returns the summatation based on the given objects
    - it takes 3 paramemters, the first two denote the range of element, and the third one is the initial value for the summation
    * there must be a conversion between the third element and the element of the collection
- `std::equal()` returns `true` if the second sequence has the equal element as the corresponding element of the first sequence, otherwise `false`
    - it takes 3 iterators, the first two denote the range of element, and the thrid one denotes the frist element in the second sequence
    * it **assumes** that the second sequcne is at least large at the first

### Algorithms That Write Container Elements
Some algorithms writes(assigns) values to the elements in a sequence
- the point is that these algorithms **aussumes** that it's safe to write the given range of the sequence
- `std::fill()` assigns the given value to each element of the given range
    * it takes 3 parameters, the first two denote the range of element, the third one is the value to assign to  
- one way to ensure that it's safe for the write algorithm to access is to use an **insert iterator**
    * for now, the simple use of `std::back_inserter` is covered
        + `std::back_inserter` is a function defined in the `<iterator>` header and returns the **insert iterator**
        + when we assign the value through dereferencing the **insert iterator** returned from the `std::back_inserter`, the new element with the given value is **added** at the back of the collection because it calls `std::push_back()` when it's dereferenced to assign to
        ```c++
        std::vector<int> vec;
        auto it = std::back_inserter(vec);
        std::fill_n(it, 10, 0);     // vec would have 10 int elements with value 0
        ```

### Algorithms That Reorder Container Elements
Some algorithms rearrange the order of elements within a container, like `std::sort()`
- when you want to eliminate the **duplicated** elements, you need to use `std::unique()` function
    * it reorders the given container so that the unique elements appear in the first part
        + but note that algorithms cannot perform container operations, the duplicated elements **remain** at the second part
        + in order to remove them, you need to call container opeartion like `.erase()`
    * it returns the iterator which denotes that first element of the second part
        + hence, you can eliminate the deplicated elements like this:
        ```c++
        void elimDups(std::vector<std::string> & words) {
            std::sort(words.begin(), words.end());
            auto it = std::unique(words.begin(), words.end());
            words.erase(it, words.end());
        }
        ```
- the point is that the library algorithms operate on iterators, not containers
    * therefore, an algorithm **cannot** (**directly**) add or remove elements

## Customizing Operations
There are some cases when we need to override the default behavaior of the opeartions

### Passing a Function to an Algorithm
As one example, assume that we want to reorder the `std::vector` of `std::string` by their size, not alphabetically
- in this case, we need to override the `<` operator of `std::sort()` by passing the third argument that is a **predicate**
    * a **predicate** is an expression that can be **called** and that **returns** a value that can be used as a **condition**
        + the predicates used by library algorithms are either **unary predicates**(meaning they have a **single** parameter) or **binary predicates**(meaning they have **two** parameters)
        + also, it must be possible to **convert** the element type to the parameter type of the predicate
    * if you want to maintain alphabetic order among the elements that have the same length, try to use `std::stable_sort()`
    ```c++
    bool isShorter(const std::string & lhs, const std::string & rhs) {
        return lhs.size() < rhs.size();
    }
    ... // some codes
    std::sort(words.begin(), words.end(), isShorter);   // passing the name of the existing function as a predicate
    std::stable_sort(words.begin(), words.end(), isShorter); 
    ```

### Callable Objects
An object or expression is **callable** when if we can apply the `()call operator` on it
- there are 4 callables
    * functions and function pointers
    * classes that overload the function call operator
    * **lambda expressions**

### Lambda Expressions
A lambda expression can be thought of as an **unnamed**, **inline** function [**object**](https://sadoe3.github.io/cpp/primer-chapter14/#lambdas-are-function-objects)
- a lambda expression has the form :
    * `[capture list] (parameter list) -> return type { function body }`
    * where `[capture list]` is an (often empty) list of **local variables** defined in the enclosing function
    * `(parameter list) -> return type { function body }` are the same as in any ordinary funciton
    * unlike a function, lambdas may be defined where **parameter** is expected
    * we can **omit** either or both of the `parameter list` and `return type` but must **include** the `capture list` and `function body`
        + the `return type` is **inferred** from the type of the expression that is returned
        + if it doesn't return anything, the `return type` is `void`
- a lambda may not have default arguments
    * therefore, a call to a lambda always has as many arguments as the lambda has parameters
```c++
// this is the revised version of the above codes
std::sort(words.begin(), words.end(), [] (const std::string & lhs, const std::string & rhs) { return lhs.size() < rhs.size(); });
```
- although a lambda may appear inside a function, it can use the variables local to that funciton **only** if it specifies them in its `capture list`
    * when you capture the local objects, there are 2 ways
        + **Capture by Value** : captured by **copying** the local object
            * `[localVar]`
        + **Capture by Reference** : captured by **referencing** the local object    
            * `[&localVar]`
            * when you capture a variable by reference, you must ensure that the variable **exists** at the time that the lambda executes
        + the difference between the two is the same as the differnce between call by value and reference
    * if you want to capture **every** entity from the enclosing function, try the **implicit capture**
        + `[&]` : capture every entity by reference
        + `[&, identifier_list]` : capture the object in `identifier_list` by value, the other ones are captured by reference
        + `[=]` : capture every entity by value
        + `[=, reference_list]` : capture the object in `reference_list` by reference, the other ones are captured by value
    * the **empty** capture list make the lambda use only **local `static`** objects or identifiers delcared **outside** the function directly
    * by default, a lambda may **not change** the value of an object **captured by value**
        + in order to change it, you need to make the lambda as `mutable` by placing the `mutable` keyword after the parameter list
        + `mutable` lambda may not omit the parameter list
        ```c++
        int a = 3;
        auto f = [a] () mutable {return ++a;};
        a = 0;
        auto j = f();       
        std::cout << a << " " << j;
        // result: 0 4
        ```
        + note that although lambda function makes its local copied object be modified, the actual object (`a` outside lambda) doesn't reflect it, because they are diffent objects
        + whether a variable captured by refernce can be changed depends only on whether that reference refers to a `const` or non`const` type
    * by default, if a `function body` contains any statements other than a `return`, that lambda is assumed to return `void` even if it returns a value
        + hence, you must explicitly include `return type` if the `function body` contains any statements other than a `return`
        ```c++
        std::transform(vi.begin(), vi.end(), vi.begin(), [] (int i) {if (i<0) return -1; else return i; });                  // error; cannot deduce the return type 
        std::transform(vi.begin(), vi.end(), vi.begin(), [] (int i) -> int {if (i<0) return -1; else return i; });         // ok
        ```

### bind Function
You can create a new **callable** objects with some parameters bound to certain values or objects by calling `std::bind()` function which is defined in `<functional>` header
```c++
#include <functional>
void printAll(int a, int b, int c, int d, int e) {
    std::cout << a << " " << b << " " << c << " " << d << " " << e << std::endl;
}
...         // some codes
auto newCallable = std::bind(printAll, 1, 2, 3, std::placeholders::_2, std::placeholders::_1);
newCallable(8, 9);      // print 1 2 3 9 8
```
- if you want to pass an object to bind as a reference, you must use the library function `std::ref` or `std::cref` defined in `<functional>` header as well


## Revisiting Iterators
The library defines several **additional** kinds of iterators in the `<iterator>` header
- **insert iterators** : these iterators are bound to a container and can be used to insert elements into the container
- **stream iterators** : these iterators are bound to input or output streams and can be used to iterate through the associated IO stream
- **reverse iterators** : these iterators move backward, rather than forward. the library containers, other thans `std::forward_list`, have reverse iterators 
- **move iterators** : these special-purpose iterators move rather than copy their elements. they would be covered in the later chapter

### Insert Iterators
There are 3 kinds of inserters
- `std::back_inserter` creates an iterator that uses `.push_back()`
- `std::front_inserter` creates an iterator that uses `.push_front()`
- `std::inserter` creates an iterator that uses `.insert()`
    * therefore, it takes a second argument which is an iterator which denotes a certain element of the container so that the new elements are inserted ahead of the element denoted by the second argument
- note that there's only one useful operation for insert iterators
    * `it = t` inserts the value `t` at the current position denoted by `it`
        + `.push_back()`, `.push_front()`, or `.insert()` is called based on the type of the iterator
        + after the insertion, the element which the iterator denotes is changed so that the concept can be maintained
            * for the iterators returned from `std::inserter`, the iterator is incremented after inserting one element so that it can denote the same element as before  
    * other operations, such as `*it`, `++it`, exist but do nothing to `it`. they return `it`

### iostream Iterators
Even though the `iostream` types are not containers, there are iterators that can be used with objects of the IO types
- `std:istream_iterator` reads an input stream

||`std::istream_iterator` Operations|
|:---:|:---|
|`istream_iterator<T> in(is);`|`in` reads values of type `T` from input stream `is`|
|`istream_iterator<T> end;`|off-the-end iterator for an `istream_iterator` that reads values of type `T`|
|`in1 == in2`|`in1` and `in2` must read the same or compatible type to each other. they are equal if they are both the end value or are bound to the same input stream|
|`in1 != in2`|the opposite case of `in1 == in2`|
|`*in`|returns the value read from the stream|
|`in->mem`|synonym for (*in).mem|
|`++in, in++`|reads the next value from the input stream using the `>>` operator for the element type. as usually, the prefix version returns a reference to the incremented iterator. the postfix version returns the old value|

- the above shows operations of `std::istream_iterator`
    * note that `std::istream_iterator` may not read the stream immediately
    * in most cases, it makes no significnat difference
        + however, if we create an `std::istream_iterator` that we destroy without using or
        + if we are synchronizing reads to the same stream from two different objects
        + then, we might care a great deal when the read happens
```c++
// example of using istream_iterator
std::istream_iterator<int> in_iter(std::cin);
std::istream_iterator<int> eof;
while(in_iter!=eof) {
    vec.push_back(*in_iter++);
}
```
- `std:ostream_iterator` writes an output stream 

||`std::ostream_iterator` Operations|
|:---:|:---|
|`ostream_iterator<T> out(os);`|`out` writes values of type `T` to output stream `os`|
|`ostream_iterator<T> out(os, d);`|`out` writes values of type `T` followed by `d` to output stream `os`. `d` points to a null-terminated character array|
|`out = val`|writes `val` to the `ostream` to which `out` is bound using the `<<` operator. `val` must have a type which is compatible with the type that `out` can write|
|`*out, ++out, out++`|these operations exist but do nothing to `out`. they return `out`|

```c++
// example of using ostream_iterator
std::ostream_iterator<int> out_iter(std::cout, "\n");
for (auto curElement : vec) {
    out_iter = curElement;
}
```

### Reverse Iterators
A reverse iterator is an iterator that traverses a container **backward**
- we can define a reverse iterator only from an iterator that supports `--`
 as well as `++`.
    * therefore, it's not possible to create a reverse iterator from a `std::forward_list` or a stream iterator
- we can obtain a reverse iterator by calling the `rbegin`, `rend`, `crbegin`, `crend` functions
- if you want to get the normal (forward) iterator which denotes to the same element as the reverse denotes to, call `.base()` method on the reverse iterator


## Structure of Generic Algorithms

### The Five Iterator Categories
The iterator operations required by the generic algorithms are grouped into five **iterator categories**

||Iterator Categories|
|:---:|:---|
|Input iterator|read, but not write; single-pass through the sequence, increment only|
|Output iterator|write, but not read; single-pass through the sequence, increment only|
|Forward iterator|read and write; multi-pass, increment only|
|Bidirectional iterator|read and write; multi-pass, increment and decrement|
|Random-access iterator|read and write; multi-pass, full iterator arithmetic|

### Algorithm Parameter Patterns
Most of the algorithms have one of the following four forms:
```c++
alg(beg, end, other args);
alg(beg, end, dest, other args);
alg(beg, end, beg2, other args);
alg(beg, end, beg2, end2, other args);
```
- note that if second range is provided, the algorithm **assumes** that it's safe to access it

### Algorithm Naming Convention
- when you pass a predicate, you can use the `overloaded` version or the `_if` version
    * the reason why there are 2 ways to do the same thing is because the `_if` version and normal version takes the **same** number arguments
    * hence, to avoid any possible ambiguities, the library provides separate named versions for these algorithms
- if you don't want to change the original sequence, try the `_copy` version of it


## Container-Specific Algorithms

### Algorithms specific to list and forward_list
Unlike the other containers, `std::list` and `std::forward_list` define several algorithms as members

||Algorithms That are Members of `std::list` and `std::forward_list`|
|:---:|:---|
||**These operations return `void`.**|
|`lst.merge(lst2)`|merges elements from `lst2` onto `lst`. Both `lst` and `lst2`|
|`lst.merge(lst2, comp)`|must be sorted. elements are removed from `lst2`. After the merge, `lst2` is empty. The first version uses the `< operator`; the second version uses the given comparison operation.|
|`lst.remove(val)`|Calls `erase` to remove each element that is `==` to the given|
|`lst.remove_if(pred)`|value or for which the given unary `predicate` succeeds.|
|`lst.reverse()`|Reverses the order of the elements in `lst`.|
|`lst.sort()`|Sorts the elements of `lst` using `<` or the given comparison|
|`lst.sort(comp)`|operation.|
|`lst.unique()`|Calls `erase` to remove consecutive copies of the same value.|
|`lst.unique(pred)`|The first version uses `==;` the second uses the given binary `predicate`.|
|`lst.splice(p, lst2)` or `flst.splice_after(p, lst2)`|`p` is an iterator to an element in `lst` or an iterator just before an element in `flst`. moves all the element(s) from `lst2` into `lst` just before `p` or into `flst` just after `p`. removes the element(s) from `lst2`. `lst2` must have the same type as `lst` or `flst` and may not be the same list|
|`lst.splice(p, lst2, p2)` or `flst.splice_after(p, lst2, p2)`|`p2` is a valid iterator into `lst2`. moves the element denoted by `p2` into `lst` or moves the element just after `p2` into `flst`. `lst2` can be the same list as `lst` or `flst`|
|`lst.splice(p, lst2, b, e)` or `flst.splice_after(p, lst2, b, e)`|`b` and `e` must denote a valid range in `lst2`. moves the elements in the given range from `lst2`. `lst2` and `lst`(or `flst`) can be the same list but `p` must not denote an element in the gien range|

- these operations return `void`
- the list member versions should be used in **precedence** to the generic algorithm for `std::list` and `std::forward_list`
- note that `list-specific` operations **do** change the container
    * becuase they are the members of the container, they change the element
    * for instance, the `list` version of `.unique()` **removes** the duplicated part

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}