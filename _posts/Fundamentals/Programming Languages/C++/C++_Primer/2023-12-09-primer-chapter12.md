---
title: "C++ Primer : 12. Dynamic Memory"

categories:
    - cpp

tags:
    - [C++, Programming Language, Dynamic Allocation, new, delete, memory]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-9
---

# Dynamic Memory

> 이 포스트는 C++ Primer(5th Edition)를 바탕으로 작성되었습니다.

## Dynamic Memory and Smart Pointers
Like C, it's possible to perform **dynamic allocation** in C++

### new and delete Operators
In C++, dynamic memory is managed through a pair of operators:
- `new`
    * which allocates, and optionally initializes, an object in dynamic memory and returns a `pointer` to that object
    * the dynmaically allocated object is in the heap area(free store)
- `delete`
    * which takes a pointer to an object dynamically allocated, destroys that object, and frees the allocated memory; returns `void`

### Smart Pointers
In C++11, the library provides 2 **smart pointer** types that manage dynamic objects
- a smart pointer acts like a regular pointer with the important exception that it executes the `delete` operator on the object to which it points when it's destroyed
    * `std::shared_ptr` allows **multiple** pointers to refer to the same object
        + `std::weak_ptr` is a companion class that is a weak reference to an object managed by a `std::shared_ptr`
    * `std::unique_ptr` allows only **single** pointer to refer to an object (as if owning it)
- All three are defined in the `<memory>` header

### Common Smart Pointer Operations

||Operations Common to `std::shared_ptr` and `std::unique_ptr`|
|:---:|:---|
|`shared_ptr<T> sp`|default construction; points to `nullptr`|
|`unique_ptr<T> up`|default construction; points to `nullptr`|
|`p`|uses `p` as a condition; `true` if `p` points to an object.|
|`*p`|dereferences `p` to get the object to which `p` points.| 
|`p->mem`|synonym for `(*p).mem`.| 
|`p.get()`|returns the pointer in `p`. use with caution, the object to which the returned pointer points will disappear when the smart pointer deletes it.| 
|`swap(p, q)`|swaps the pointers in `p` and `g`.| 
|`p.swap(q)`|equivalent to `swap(p, q)`| 

- smart pointers are **templates** which means we must supply additional information
    * which is the type to which the pointer can point
- a default initialized smart poiner holds a `nullptr`
- dereferencing returns the object to which the pointer points
- if it's used as a condition, the effect is to test whether the pointer is `nullptr` or not

### make_shared Function
The safest way to allocate and use dynamic memory is to call a library function named `std::make_shared` defined in the `<memory>` header
```c++
#include <memory>
std::shared_ptr<int> p1 = std::make_shared<int>();  // value-initialized
```
- this function allocates and initializes an object in dynamic memory and returns a `std::shared_ptr` that points to that object
- because it's a function template, you need to specify the type as the class argument
- like `.emplace()`, `std::make_shared` uses its argument to **construct** (not copy) an obejct of the given type

### Copying and Assigning shared_ptr
We can think of a `std::shared_ptr` as if it has an associated counter, usually referred to as a **reference count**
- whenever we **copy** a `std::shared_ptr`, the count is **incremented**
    + when we use the existing `std::shared_ptr` to copy-contruct another `std::shared_ptr`
    + or, we use the existing `std::shared_ptr` as the right-hand operand of an assignment
    + or, when we pass the existing `std::shared_ptr` to or return it from a function by value
    + the count is incremented
- whenever the `std::shared_ptr` **doesn't point** to the initial address, the count is **decremented**
    + when we assign a new address to the exising `std::shared_ptr`
    + or, when the exising `std::shared_ptr` itself is destroyed, such as when a local `std::shared_ptr` goes out of scope
- once a `std::shared_ptr`'s count goes to zero, the `std::shared_ptr` automatically frees the objects that it manages

### shared_ptr Operations

||Operations Specific to `std::shared_ptr`|
|:---:|:---|
|`make_shared<T>(args)`|returns a `std::shared_ptr` pointing to a dynamically allocated object of type `T`. Uses `args` to initialize that object.|
|`shared_ptr<T> p(q)`|`p` is a copy of the `std::shared_ptr` `g`; increments the count in `g`. The pointer in `g` must be convertible to `T*`.|
|`shared_ptr<T> p(q)`|`p` manages the object to which the built-in pointer `q` points; `q` must point to memory allocated by new and must be convertible to `T*`.|
|`shared_ptr<T> p(u)`|assumes ownership from the `std::unique_ptr` `u`; makes `u` null.|
|`shared_ptr<T> p(q, d)`|assumes ownership for the object to which the built-in pointer `q` points. `q` must be convertible to `T*`. `p` will use the callable object `d` in place of `delete` to free `q`.|
|`shared_ptr<T> p(p2, d)`|`p` is a copy of the `std::shared_ptr` `p2` except that `p` uses the callable object `d` in place of `delete`.|
|`p = q`|`p` and `q` are `std::shared_ptr`s holding pointers that can be converted to one another. Decrements `p`'s reference count and increments `q`'s count; deletes `p`'s existing memory if `p`'s count goes to `0`.|
|`p.use_count()`|returns the number of objects sharing with `p`; may be a slow operation, intended primarily for debugging purposes.| 
|`p.unique()`|returns `true` if `p.use_count()` is `1`; `false` otherwise.| 
|`p.reset()`|if `p` is the only `std::shared_ptr` pointing at its object, `reset` frees|
|`p.reset(q)`|`p`'s existing object. if the optional built-in pointer `q` is passed,| 
|`p.reset(q, d)`|makes `p` point to `q`, otherwise makes `p` null. if `d` is supplied, will call `d` to free `q` otherwise uses `delete` to free `q`.|

- note that it's possible to supply **custom** `deleter` function which would be called instead of `delete` operator for freeing the pointer 

### How to Use new Operator
The following codes shows the basic syntax of the `new` oeprator
```c++
Type *ptr = new Type;                   // default initialized
Type *ptr = new Type();                 // value initialized
Type *ptr = new Type(args);             // initialized with the proper contructor based on args
Type *ptr = new Type{ a, b, c, ...};    // list initialized

auto p1 = new auto(obj);                // ok; p1 points to an object of the type obj 
                                        // which is initialized from obj
auto p2 = new auto{a,b,c};              // error; must use parentheses for this initializer

const Type *ptr = new const Type(args);         // const object can be allocated;
                                                // must be initialized
Type * p1 = new (std::nothrow) Type;            // handling memory exhaustion
```
- by default, if `new` is unable to allocate the request storage, it throws an exception of type `std::bad_alloc`
    * if you want to prevent `new` from throwing an exception, try to use **placement new** which lets us pass additional arguments to `new`
    * if you pass `std::nothrow` to `new`, `new` will return a `nullptr` if the allocation failed

### How to Use delete Operator
The following codes shows the basic syntax of the `delete` oeprator
```c++
delete p;           // p must point to a dynamically allocated object or be nullptr


int i, *pi1 = &i, *pi2 = nullptr;
double *pd = new double(33), *pd2 = pd;
const int *pci = new const int(1024);

delete i;           // error; i is not a pointer
delete pi1;         // undefined; pi1 refers to a local object
delete pd;          // ok
delete pd2;         // undefined; the memory pointed to by pd2 was already freed
delete pi2;         // ok; it's always ok to delete a nullptr 
delete pci;         // ok; deletes a const object
```
- the basic syntax of the `delete` operator is quite simple, however the important point is that
    * you must pass the **proper pointer** to the `delete` operator
    * because compiler checks **only** whether it's a pointer or not
        + if you pass a pointer to a local object
        + or, pass a pointer which points to the memory already freed
        + then, the result is **undefined** 

### Core Concept of Dynamic Allocation
If you want to control the dynamic allocation **directly**
- you need to ensure the following list
    * do not forget to call `delete` operator after using the dynamic object
    * reset the pointer as `nullptr` after calling `delete` operator
        + reset every pointer which points to the same memory when one of them is freed
- if you are not confident regarding this, then just use **smart pointer**
- do not mix ordinary pointers and smart pointers because it's error-prone

### Using Smart Pointers with new
The smart pointer contstructors that take pointers are `explicit`, therefore you must use the direct form of initialization
```c++
std::shared_ptr<int> p1 = new int(1024);       // error;
std::shared_ptr<int> p2(new int(1024));        // ok;
```

### Smart Pointers and Exceptions
The smart pointer class ensures that memory is freed when it is no longer needed even if the block is exited **prematurely**
```c++
void f() {
    std::shared__ptr<int> sp(new int(42));
    ... // exception thrown; not caught inside f()
}   // shared_ptr freed automatically when the function f() ends
```
- by default, ordinary pointers don't support this

### Conventions of Smart Pointers
To use smart pointers correctly, we msut adhere to a set of the following conventions
- don't use the same built-in pointer value to initialize (or reset) more than one smart pointer
- don't `delete` the pointer returned from `.get()`
- don't use `.get()` to initialize or reset another smart pointer
- if you use a pointer returned by `.get()`, remember that the pointer will become invalid when the last corresponding smart pointer goes away
- if you use a smart pointer to manage a resource other than memory allocated by `new`, remember to pass a deleter

### unique_ptr Operations

||Operations Specific to `std::unique_ptr`|
|:---:|:---|
|`make_unique<T>(args)`|returns a `std::unique_ptr` pointing to a dynamically allocated object of type `T`. use `args` to initialize that object.|
|`unique_ptr<T> p`|null `std::unique_ptr`s that can point to objects of type `T`. `p` will use `delete` to free its pointer|
|`unique_ptr<T> p(q)`|`p` manages the object to which the built-in pointer `q` points|
|`unique_ptr<T, D> p`|null `std::unique_ptr`s that can point to objects of type `T`. `p` will use a callable object of type `D` to free its pointer|
|`unique_ptr<T, D> p(d)`|null `std::unique_ptr` that point to objects of type `T` that uses `d`, which must be an object of type `D` in place of `delete`.|
|`p = nullptr`|deletes the object to which `p` points; makes `p` null.|
|`p.release()`|relinquishes control of the pointer `p` had held; returns the pointer `p` had held and makes `p` null.|
|`p.reset()`|deletes the object to which p points;|
|`p.reset(nullptr)`|equivalent to `p.reset()`|
|`p.reset(q)`|if the built-in pointer `q` is supplied, makes `p` point to that object. otherwise makes `p` null.|

- by default, we **cannot** perform **copy construction** from the existing `std::unique_ptr` nor **assign** the existing `std::unique_ptr` to another one
    * there is one exception to this rule
    * we can copy or assign a `std::unique_ptr` that is about to be destroyed
    * the most common example is when we return `std::unique_ptr`from a function
    ```c++
    std::unique_ptr<int> doCopy(int a) {
    return std::make_unique<int>(a);
    }
    ...                                 // some codes
    std::unique_ptr<int> a;
    a = doCopy(3);
    std::cout << *a << std::endl;       // prints 3
    ```
- if you want to move the ownership, we can perform `std::move` operation on `std::unique_ptr` objects
- like `std::shared_ptr`, it's possible to provide custom `deleter` function for `std::unique_ptr` 

### Exceptions regarding unique_ptr
Note that if you construct two or more `std::unique_ptr` with the same address, the **run time error** occurs
- the point is there's **no exception** thrown in this case
- therefore, if you want to make the robust program, you need to ensure that
    * `std::unique_ptr`s are constructed by only either `std::make_unique` or `new` operator
        + not the ordinary pointer which points to the memory
```c++
int* ptr = new int(3);
std::unique_ptr<int> up1(ptr);          // ok
std::unique_ptr<int> up2(ptr);          // run time error
```  

### weak_ptr
A `std::weak_ptr` is a smart pointer which does not control the lifetime of the object which it points to because it's managed by `std::shared_ptr`
- `std::weak_ptr` is constructed or assigned by `std::shared_ptr` so that both of them point to the same object
    * however, adding or removing `std::weak_ptr` **doesn't change** the **reference count** of the `std::shared_ptr`
    * henece, it may happen that `std::weak_ptr` is invalidated when all `std::shared_ptr` which point to the same object are gone although it's alive

### weak_ptr Operations

||Operations Specific to `std::weak_ptr`|
|:---:|:---|
|`weak_ptr<T> w`|null `std::weak_ptr` that can point at objects of type `T`.|
|`weak_ptr<T> w(sp)`|`std::weak_ptr` that points to the same object as the `std::shared_ptr` `sp`. `T` must be convertible to the type to which `sp` points.|
|`w = p`|`p` can be a `std::shared_ptr` or a `std::weak_ptr`. after the assignment `w` shares ownership with `p`.|
|`w.reset()`|makes `w` null.|
|`w.use_count()`|returns The number of `std::shared_ptr`s that share ownership with `w`.|
|`w.expired()`|returns `true` if `w.use_count()` is `0`, `false` otherwise.|
|`w.lock()`|if `w.expired()` is `true`, returns a null `std::shared_ptr`; otherwise returns a `std::shared_ptr` to the object to which `w` points.|


## Dynamic Array
Like C, it's possible to allocate many objects at once in C++
- there are 2 ways to implement it
    * using `new []` and `delete []`
        + **couples** allocation and initialization
        + also, destruction and deallocation
    * using `std::allocator` class
        + **decouples** allocation and initialization
        + also, destruction and deallocation

### new and delete and Arrays
The following codes shows how to allocate a dynamic array using `new []` and `delete []`
```c++
// basic syntax of new []
Type *ptr = new Type[size];         // default initialized
Type *ptr = new Type[size]();       // value initialized
Type *ptr = new Type[size] {a, b, c, Type(aa, bb), ...};         // braced list initialized
Type *ptr = new Type[0];            // ok; but ptr can't be dereferenced

// basic syntax of delete []
delete [] ptr;                      // brackets are necessary when deallocating dynamic array 
```
- although it's common to refer to memory allocated by `new T[]` as a **dynamic array**, this usage is somewhat **misleading**
    * because it returns the type of the **element**, not the array type
    * hence, we can't call `std::begin` or `std::end` on a dynamic array
    * which means, we can't use a range `for` to iterate element in a (so-called) dynamic array
- if there are more initializers than the given size, the `new` expression fails and no storage is allocated
    * in this case, `new` throws an excpetion of type `std::bad_array_new_length` defined in the `<new>` header
- unlike normal array, it's possible to allocate an array of size zero
    + `new` returns a valid, nonzero pointer which acts as the off-the-end pointer for a zero-element array
- although `delete []` operator looks simple
    * if you omit `[]`, the result is **undefined**

### Smart Pointers and Dynamic Arrays
It's possible for smart pointer to point to the dynamic array, however, only `std::unique_ptr` partially supports it
```c++
// unique_ptr
std::unique_ptr<int []> up(new int[10]);    // must include []
for(int i = 0; i != 10; i++)
    up[i] = i;                              // support indexing only

// shared_ptr
std::shared_ptr<int> sp(new int[10], [] (int*p) { delete [] p; });      // lambda is provided as the deleter function
for(int i = 0; i != 10; i++)
    *(sp.get() + i) = i;                    // primitive way to implement indexing
```
- if you include `Type []` as the template argument for `std::unique_ptr`, it would call `delete []` for destruction and deallocation
    * however, `std::shared_ptr` doesn't support this, which means you have to supply the `deleter` function which performs `delete []` for the dynamic array
- `std::unique_ptr` supports `[]` operator only
    * it doesn't support `.` or `->` operators
    * however, `std::shared_ptr` supports nothing
        + we need to perform very primitive way to implement indexing for `std::shared_ptr` with dynamic array

### The allocator Class
The library `std::allocator` class, which is defined in the `<memory>` header, lets us separate allocation from construction

||Operations Specific to `std::allocator` class|
|:---:|:---|
|`allocator<T> a`|defines an allocator object named `a` that can allocate memory for objects of type `T`.|
|`a.allocate(n)`|allocates raw, unconstructed memory to hold `n` objects of type `T`.|
|`a.construct(p, args)`|`p` must be a pointer to type `T` that points to raw memory; `args` are passed to a constructor for type `T`, which is used to construct an object in the memory pointed to by `p`.|
|`a.destroy(p)`|runs the destructor on the object pointed to by the `T*` pointer `p`.|
|`a.deallocate(p, n)`|deallocates memory that held `n` objects of type `T` starting at the address in the `T*` pointer `p`; `p` must be a pointer previously returned by `.allocate()`, and `n` must be the size requested when `p` was created. the user must run `.destroy()` on any objects that were constructed in this memory before calling `.deallocate()`.|
||**These functions construct elements in the destination, rather than assigning to them; they are defined in the `<memory>` header** |
|`uninitialized_copy(b, e, b2)`|copies elements from the input range denoted by iterators `b` and `e` into unconstructed, raw memory denoted by the iterator `b2`. the memory denoted by `b2` must be large enough to hold a copy of the elements in the input range.|
|`uninitialized_copy_n(b, n, b2)`|copies `n` elements starting from the one denoted by the iterator `b` into raw memory starting at `b2`.|
|`uninitialized_fill(b, e, t)`|constructs objects in the range of raw memory denoted by iterators `b` and `e` as a copy of `t`.|
|`uninitialized_fill_n(b, n, t)`|constructs an unsigned number `n` objects starting at `b`. `b` must denote unconstructed, raw memory large enough to hold the given number of objects.|

- because it's a template, we need to specify the type
- using unconstructed memory is **undefined** except for construction 
- the following shows the example of using the `std::allocator` class
    ```c++
    constexpr int SIZE = 3;
    std::allocator<int> alloc;

    auto ptr = alloc.allocate(SIZE);
    const auto START = ptr;
    for (signed index = 0; index != SIZE; index++, ptr++)
        alloc.construct(ptr, index);
    ptr = START;

    for (signed index = 0; index != SIZE; index++)
        std::cout << ptr[index] << " ";                 // prints 0 1 2
    std::cout << std::endl;

    for (signed index = 0; index != SIZE; index++, ptr++)
        alloc.destroy(ptr);
    alloc.deallocate(START, SIZE);
    ```

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}