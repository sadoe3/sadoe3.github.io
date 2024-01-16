---
title: "C++ Primer : Chapter 6"

categories:
    - cpp

tags:
    - [C++, Programming Language, Function]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-2
---

# Functions

> 이 포스트는 C++ Primer(5th Edition)를 바탕으로 작성되었습니다.

## Prerequisites regarding functions
- this post assumes that you've already studied about functions through learning C
- hence, this post covers only about how the C++ features work with the existing concepts of functions

### C++ features
- passed by value and passed by refernece - p208
- reference to array (p114) - p 217
- varing parameters - p 220
- return list-intialzation - p226
- trailing return type and decltype as return type - p229
- overloaed functions - p230
- default argument - p236
- inline and constexpr - p238
- function matching - p242


## Argument Passing
- Passed by value and Passed by reference
- Varing Parameters

### Passed by value and Passed by reference
C and C++ have difference concepts regarding passing an argument as passed by value or passed by reference
- In **C**, passing a pointer is the sort of **passed by reference** because you **can change its value** through dereference it
    * From the perspective of C++, there's no way in C to pass an argument as **passed by reference** because there's no reference type in C
- In **C++**, however, passing a pointer is the sort of **passed by value** because it's not a reference so that **it's copied**
    * In C++, the only way to pass an argument as **passed by reference** is to make the parameter as a reference
    * In this way, the initializer is **not copied** because the parameter is a reference
- If you want to use a parameter as a reference in order not to copy the initializer and don't want to change its value
    * try to make the parameter as **reference to const**
- reference argument can be used as additional return values
    ```c++
    int factorial(int &countCalled, int number) {
        countCalled++;
        if (number == 1)
            return 1;
        return number * factorial(countCalled, number - 1);
    }
    ```
- you can set the parameter as a refence to an array
    ```c++
    int func(int (&rArr)[10]) {
        std::cout << rArr[2] << std::endl;
    }
    ```

### Varing Parameters
You can implement a function which takes **varing number** of arguments in 2 ways
- C style
    * using **ellipsis**
    * same rules applied in C++ as in C
    * use this feature only if you need to interface to C code otherwise, try C++ style features
- C++ style
    * varing number of argument of the **same** type : `initializer_list<type>` template
    * varing number of argument of the **different** type : [**variadic template**](https://sadoe3.github.io/cpp/primer-chapter16/#variadic-templates)

### initializer_list\<type>
```c++
#include <initializer_list>

int function(double a, std::initializer_list<int> b) {
    std::cout << b.size() << std::endl;

    std::initializer_list<int> c = b;       // doesn't copy, c and b share the element
    for(auto cur = b.begin(), end = b.end(); cur != end; cur++) {
        std::cout << *cur << " " << std::endl;
    }
}

... // some codes
function(3.14, {1, 2, 3, 4, 5});        // list-intialization
```
- you must include `<initializer_list>` to use `initializer_list<type>` template
- there are 2 ways to intialize it
    * default-initialization
    * list-initialization
- when `intializer_list<type>` object is copied or assigned to another object, it's not acutally copied
    * they just share the elements
- there are only a few operations available in `intializer_list<type`
    * `.size()`
    * `.begin()` and `.end()`
- you can put other parameters on the same function which takes `initializer_list<type>` parameter but you need to put the `initializer_list<type>` at the last part of the parameter list like ellipsis


## Return Types
You can set the deduced type as a return type of a function

### Trailing Return Type
```c++
auto functionName(int a) -> int(*)[10] {    // returns a pointer to an array of 10 int elements
    ... // some codes
}
```
- when you set auto as a return type, you can specify the actual return type by using trailing return type
- this is useful when the return type is complicated like above codes

### decltype as a return type
```c++
int odds[3] = {1,3,5};

decltype(odd) * arrPtr(int i) {     // returns a pointer to an array of 3 int elements
    return &odd;
}
```
- like auto, `decltype` can be used as a return type


## Features for Specialized Uses

### Default Arguments
It's possible to set the default value to the parameter in C++ like C but in a differnt way
```c++
int functionName(int, int = 3, int = 6);

int functionName(int, int, int = 3);  // error; you can't change the default value
int functionName(int = 1, int, int);  // ok; you can add default arguments
```
- you can set the default value by using = operator to the parameter
- like C, you must put the parameters with default values to the last part of the parameter list
- you can add more default arguments but cannot change the default value
- objects can be used as the default value of the parameters
    ```c++
    char c = 'k';
    int a = 4;
    void functionName(int = a, char = c);       // think as if they are bound

    void f2() {
        c = 'z';
        int a = 5;          // hides the global variable but doesn't change the default value
        functionName();     // calls functionName(4, 'z')
    }
    ```

### inline Functions
If simple operation is used in many places, making it as a function is a good idea, and making such function as an `inline` is also a good idea to increse the performance
```c++
inline const std::string & shorterString(const std::string &s1, const std::string &s2) {
    return s1.size() <= s2.size() ? s1 : s2;
}
```  
- placing `inline` before the return type defines the function as an `inline`
- when a function is defined as an `inline`, the function call **requests** the compiler to replace itself to its definition
    * note that the **request** can be ignored when the body is too long

### constexpr Functions
`constexpr` function can be called in the **constant expression**
```c++
constexpr int getNumber() { return 3; }                                 // getNumber() is constexpr function
constexpr size_t scale(size_t count) { return count*getNumber(); }      // scale(size_t count) is constexpr function when count is constant expression  
```
- placing `constexpr` before the return type defines the function as a `constexpr` function
- however, there are 2 conditions to be satisfied in order to make the function as a `constexpr` function
    * return type, and paramter types must be a **literal type**
        + if the argument is given at run time, this doesn't satisfy this condition
    * function bojdy must contain only **one** `return` statement
- if the function satisfies the conditions, then the compiler would replace the function call to its **resulting value**
    - hence `constexpr` functions are implicitly `inline` functions
- it's usually a good idea to put `inline` and `constexpr` functions in the header file



## Function Overloading
We can implement **funciton overloading** in a different way from C in C++
```c++
void print(const char *cp);
void print(const int *begin, const int *end);
void print(int ia[], size_t size);

... // some codes
int j[2] = {0,1};
print("hello world");           // call print(const char*)
print(j, end(j) - begin(j));    // call print(int*, size_t)
print(begin(j), end(j));        // call print(const int*, const int*)
```
- functions which have the **same name** and **scope** but **different parameters** are **overloaded**
    * functions which have the same names but differnt scopes are not overloaded
    ```c++
    int func(int);
    int main() {
        int func(char *);
        func(3);        // error because func(int) is hidden due to the local function
    }
    ```
    * in C++, name lookup happens before type checking
- we can make the function of a certain name work differently by passing different arguments
    * this is the core concept of the **function overloading**
    * but if the functions work in completely different ways, then just using different names might be a better solution
- like a copy operation, compiler ignores top-level const and respects low-level const
    * functions have the different parameters in terms of top-level const only are regarded as the same functions

### Function Matching
**Function Matching** is the process to find the proper function to call in the overloaded set
- there are 3 possible results
    * best match : there's only one appropriate function
    * no match : compilation error when there's no proper fucntion available
    * ambiguous call : compilation error when there are more than one functions which can be a proper one
    * ambiguous call happens when there are functions which takes the same number of arguments and the types are different from the call but can be converted
- the process consists of the 3 phases
    1. find **candidate** functions
        * candidate functions have the same name and are visible from the caller
    2. find **viable** functions
        * viable functions have the same number of parameters with proper types which may be converted
    3. find the **best** function
        * the best case is to find the function which has the parameters of exactly same types
        * if it's not the best case, then compiler finds **better** function by comparing the type conversions
        * if it's not possible to find the best function, then it is the **ambiguous call** which is a compilation error
        ```c++
        int functionName(int, int);
        int functionName(double, double);        
        ... // some codes
        function(3.14, 3);      // ambiguous call
        // because int functionName(int, int) is better in terms of the first argument, but int functionName(double, double) is better in terms of the second argument 
        ```