---
title: "Effective C++ : Chapter 8"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective C++, Dynamic Allocation, new handler]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-31
---

# Customizing new and delete

> 이 포스트는 Effective C++(3rd Edition)를 바탕으로 작성되었습니다.

## Item 51

### Adhere to convention when writing new and delete
The code below is the pseudocode for a non-member operator `new`
```c++
void * operator new(std::size_t size) throw(std::bad_alloc) {
    using namespace std;

    if (size == 0)
        size = 1;       // handle 0-byte requests by treating them as 1-byte request

    while (true) {
        attempt to allocate size to bytes;
        if(the allocation was successful)
            return (a pointer to the memory);

        // allocation was unsuccessful; find out what the current
        // new-handling function is
        new_handler globalHandler = set_new_handler(0);
        set_new_handler(globalHandler);

        if(globalHandler)
            (*globalHandler)();
        else
            throw std::bad_alloc();
    }
}
```
- note that the operator `new` needs to return non-null pointer although it takes 0-byte request
- the code above said if the allocation failed
    * then, it checks whether the pointer to new-handler is `nullptr` or not
    * if it's not `nullptr`, then it calls that new-handler
        + oterwise, it throws `std::bad_alloc` excaption
- you can specify the new-handling function by calling `std::set_new_handler()`
    * which takes a pointer to function which takes nothing and returns nothing
    * and `std::set_new_handler()` returns the pointer to function which it previously contained
- note that the process above is inside **infinite loop**
    * the way out of loop is
        + for memory to be successfully allocated
        + for the new-handling function to do one of the 5 tasks below
            - Make more memory available
            - Install a different new-handler
            - Deinstall the new-handler
            - Throw an exception
            - Terminate program
                * by calling `std::abort()` or `std::exit()`
    * if there's no new-handler and it failed to allocate memory, then
        + it throws `std::bad_alloc` excaption
- the simple new-handling function can be implemented like this:
    ```c++
    void outOfMem() {
        std::cerr << "Unable to satisfy request for memory\n";
        std::abort();
    }
    ```
- operator `delete` should do nothing if `nullptr` is given


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}