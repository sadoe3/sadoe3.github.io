---
title: "C++ Primer : 3. Strings, Vectors, and Arrays"

categories:
    - cpp

tags:
    - [C++, Programming Language, STL, Vector, String, Iterator, Array]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-11-28
---

# Strings, Vectors, and Arrays

> 이 포스트는 C++ Primer(5th Edition)를 바탕으로 작성되었습니다.

## Namespace using Declaration

### using Declaration
In order to use a name inside a certain namespace, you should type like this : `namespace::name` <br>
but with **using** declaration, you can skip `namespace::` part 
```c++
#include <iostream>

using std::cin;                     // using declaration
using std::endl; using std::string; // multiple using delcarations

int main() {
    int i;

    cin >> i;       // ok
    
    cout << i;      // error
    std::cout << i; // ok

    return 0;
}
```
- `::` is the scope operator which let compiler look in the scope of the left-hand operand for the name of the right-hand operand
- when you want to use multiple using declarations in a single line, you need to separate which declaration by semicolon and put using respectively
- headers should not include using declaration
    * because source codes which includes them may not want to use that using declaration

## Library string Type

### Use the C++ version of C Libarary Headers
The C++ library incorporates the C library for facilities defined specifically for C++
```c++
#include <name.h>   // c style header
#include <cname>    // c++ style header

// example
#include <ctype.h>
#include <cctype>
```
- the c++ version of these headers are named cname - they remove the .h suffix and preced the name with the letter c
- in particular, the names defined in the cname headers are defined inside the std namespace, whereas those defined in the .h versions are not

### STL and sequential container
In this chapter, string and vector which are sequential container of STL are covered in order to know how to use them in basic way
- STL would be covered deeply in later chapter

### std::string
`std::string` is a class which handles the **variable-length** sequence of characters 
```c++
#include <string>
... // some codes

// basic initialiation of std::string
std::string s1;             // default initialization
std::string s2 = s1;        // s2 is the copy of s1
std::string s2(s1);         // same as std::string s2 = s1; 
std::string s3 = "value";   // s3 is the copy of string literal not including null character
std::string s3("value");    // same as std::string s3 = "value"; 
std::string s4(n, 'c');     // s4 is initialized with n copies of character 'c'
```
- in oder to use string class, you need to include \<string> header file
- because string class is defined in std namespace, you need to specify std:: to using string class
    * if you don't want this, try to use using declaration : `using std::string;`
- if string is created by default initialization, it becomes an empty string
- when you intialize an object, there are 2 types based on the existence of =
    * if there is = for initialization, then it would be **copy initialization**
        + through this type of initialization, the object is a copy of the initializer
    * f there is not = for initialization, then it would be **direct initialization**
        + through this type of initialization, the object is initialized by the implemented way of class author


### Operations on strings

||Basic string Operations|
|:---:|:---|
|`os << s`|Writes s onto ouput stream os. Returns os.|
|`is >> s`|Reads whitespace-separated string from is into s. Returns is.|
|`getline(is, s)`|Reads a line of input from is into s. Retuns is.|
|`s.empty()`|Returns true if s is empty; otherwise returns false.|
|`s.size()`|Returns the number of characters in s.|
|`s[n]`|Returns a reference to the char at position in n in s; positions start at 0.|
|`s1 + s2`|Returns a string that is the concatenation of s1 and s2.|
|`s1 = s2`|Replaces characters in s1 with a copy of s2.|
|`s1 == s2`|Returns true if s1 and s2 contain the same characters. Equality is case-sensitive.|
|`s1 != s2`|Opposite case of ==|
|`<, <=, >, >=`|Comparison are case-sensitive and use **dictionary ordering**|

```c++
string s1, s2;

std::cin >> s1 >> s2;       // becuase std::cin>> s1 returns std::cin, we can chain multiple i/o

while(std::cin >> s1)       // read until end-of-file
    std::cout << s1 << std::endl;
```
- i/o operation
    * streams for standard i/o use white space for separator, and once it finds white space, it discards it
    * i/o stream can be used as condition, in this case, it returns true if it handles valid i/o and not end-of-file, and false for opposite case
- `getline()` function
    * similar to input operator(>>)
    * difference: getline uses new line character only as separator
        + hence, when it finds new line character, it stops reading and discards that new line character
        + if the fisrt character is new line character, it returns empty string
    * getline returns the given input stream, so getline() function can be used inside a condition
- `s.empty()` and `s.size()` method
    * becuase they are methods (functions defined inside a class), you need to use .(dot operator) to call them
    * s.size() method returns not int but std::string::size_type
        + this type is a **companion type**
        + most classes in STL have their own companion types
        + the reason why they use companion type is because they want to use their companion type in a machine-independent manner
        + if you want an object which contains the size, try to use auto instead of int
- `==` : equality operator
    * two strings are equal if
        + they have same length
        + and have same characters (case sensitive)
- `<, <=, >, >=` : relational operators
    * they use the same strategy as a dictionary
        1. if two strings have different lengths and if every character in the shorter string is equal to the corresponding character of the longer string, then the shorter string is less than the longer one
        2. if any characters at corresponding positions in the two strings differ, then the result of the string comparsion is the result of comparing the first character at which the strings differ
- `+` : concatenation
    * concatenation can be executed between string and string literals
        + but at least on operand to each + operator must be of string type
        + which means concatenation can not be done between string literals only

### How to access the character of a string
There are 3 ways to do so
- **range for** statement
    * this new statement would be covered deeply in the later chapter, hence in this chapter, basic use of it would be covered
    * if you want to access every element, try this way

        ```c++
        std::string str = "value"

        for(auto curChar : str) {
            ...     // do something
        }

        for(auto & curChar : str) {
            curChar = 'a';  
        }
        ```
    * curChar is the character of the given string str
    * range for statement iterates str from the first element to the last one
    * if you want to chnage the value of the certain character, you need to set curChar as a reference
- subscript operator( [ ] )
    * this operator just does **indexing** like using array
    * if you want to access to certain elements, try this way
    * because it's same as indexing, the valid range is 0 to size() - 1
        + the result of using an index outside this range is **undefined**
        + subscripting an empty string is also **undefined**
        + hence, when you use subscript operators, check whether the index is valid
    * therefore, it's a good choice to use `.at()` method which acts same as what `[] operator` does but it throws an exception if it takes invalid index
        + which means it can increase the robustness of your code
- iterators
    * you can access the element of containers in STL by using their iterators
    * but they would be covered in the later chapter


### std::vector
`std::vector` is a class template which handles the **variable-length** collection of objects of which have the same type
```c++
#include <vector>

// basic initialiation of std::vector
std::vector<int> iVec;              // collection of int objects; default initialization
std::vector<int> iVec2 = iVec       // copy initialization
std::vector<int> iVec2(iVec);       // direct initialization; same as iVec2 = iVec 
std::vector<double> dVec = iVec;    // error; different type 

std::vector<std::string> articles = {"a", "an", "the"};     // list initialization
std::vector<std::string> articles{"a", "an", "the"};        // equivalent to above
std::vector<std::string> articles("a", "an", "the");        // error

std::vector<std::string> sVec(10, "hi!");     // sVec is initialized with 10 objects which are initialized by the initializer of "hi!"
std::vector<std::string> sVec(10);            // sVec is initialized with 10 objects which are default-initialized
std::vector<std::string> sVec = 10;           // error; only direct initialization works
```
- templates would be covered in the later chapter, hence knowing the basic use of vector is enough for this chapter
- in oder to use vector class template, you need to include `<vector>` header file
- because vector class template is defined in std namespace, you need to specify std:: to using vector class template
- the process that the compiler uses to create classes or functions from templates is called **instantiation**
    * when you want to instantiate the class from the vector class template, you need to specifiy the type for the collection to contain by using `<>`
    * we can define vectors to hold objects of most any type
- if vector is created by default initialization, it becomes an empty collection
- because vector is a sequential container, the intialization is similar to string
- when you instantiate vector object by copying the existing vector object, they must have same type
- you can give multiple objects at once by using list initialization
    * you need to use curly braces, not just normal parentheses to use list initialization
    * parentheses might be used for initialization of other forms
- you can instantiate a vector object with multiple objects which have the same initializer
    * if the initializer is providied, then that initializer would be used
    * otherwise, the objects are create by default initialization (value initialization)
    * you need to use direct initialization only to use this type of initialization (not copy initialization)

### Operations on vectors

||Basic vector Operations|
|:---:|:---|
|`v.push_back(t)`|Adds an element with value t to end of v.|
|`v.empty()`|Returns true if v is empty; otherwise returns false.|
|`v.size()`|Returns the number of elements in v.|
|`v[n]`|Returns a reference to the element at position n in v; positions start at 0.|
|`v1 = v2`|Replaces the elements in v1 with a copy of the elements in v2.|
|`v1 = {a,b,c,..}`|Replaces the elements in v1 with a copy of the elements in the comma-separated list.|
|`v1 == v2`|Returns true if v1 and v2 have the same number of elements and each element in v1 is equal to the corresponding element in v2.|
|`v1 != v2`|Opposite case of ==|
|`<, <=, >, >=`|Comparisons have their normal meanings and use **dictioary ordering**|

```c++
string s1, s2;

std::cin >> s1 >> s2;       // becuase std::cin>> s1 returns std::cin, we can chain multiple i/o

while(std::cin >> s1)       // read until end-of-file
    std::cout << s1 << std::endl;
```
- basic use of vector
    * the vector class template is implemented in a very efficient way, hence using the vector which is created as an empty collection and grows by calling push_back() method is not a wrong way.
- `v.size()` method
    * v.size() method returns not std::vector::size_type but std::vector\<type>::size_type
        + because vector is a template, you need to specify the certain type to use its companion type
        + same concepts applied to this as string
    
### How to access the object of a vector
There are 3 ways like string
- range for statement
- subscript operator
    * range is same as string
        + 0 to v.size() -1
    * subscripting the object at the position outside valid range would return the **undefined** value
        + it does **not** add new element at that position
        + hence, you need to check the position before indexing
        + or use `.at()` method
- iterators


## Introducing Iterators

### Iterators
You can think of an iterator as the pointer which points to the element of the container of STL
- technically speaking, string is not a container, but it acts like a container
- but the explanation above is enough to understand what the iterators are  


### Operations on iterators
Because the iterators are like pointers, you can do basic pointer operations on iterators

||Basic iterator Operations|
|:---:|:---|
|`*iter`|Same dereferencing. Return a reference to the element denoted by the iterator iter.|
|`iter->mem`|Same arrow operation as pointers.|
|`++iter`|Same increment operation as pointers. postfix as well.|
|`--iter`|Same decrement operation as pointers. postfix as well.|
|`iter1 == iter2`|Same comparison. Returns true if they denote the same element or they are the off-the-end iterator for the same container.|
|`iter1 != iter2`|Opposite case of ==|

```c++
std::vector<int> v1;

for(std::vector<int>::iterator curIt = v1.begin(), endIt = v1.end(); curIt != endIt; curIt++) {
    std::cout << *curIt << std::endl;       // print all elements
}
```
- there are 2 ways to get iterators
    * calling the begin() methods
        + there are several types of begin and end() methods
        + all types of iterators would be covered in the later chapter, for now, knowing how to use basic iterator is enough
        + begin() method returns an iterator which points to the first element of the container
    * calling the end() methods
        + end() method returns an iterator which points to the one past the end of the container
        + the iterator returned by end() method often referred to as the off-the-end iterator or the end iterator
        + surely, dereferecing end iterator is undefined
        + end iterator exists to check whether accessing all elements or not because after iterator which points to the last element moves forward one step, the iterator becomes the end iterator
- because iterators support operations related to pointers, you can use them like pointers with one exception where the one after last element is the end iterator

### Iterator arithmetic
Just like pointers, there are operations called **iterator arithmetic**
```c++
std::string a = "haha";

std::string::iterator iter = a.begin();
iter++;             // now iter points to the second character
iter += 1;          // same as iter++; iter points to the third character
std::cout << *(iter-2) << std::endl;    // print the first character
*(iter+30)          // undefined;

auto iter1 = a.begin();
auto iter2 = a.begin();
iter2++;
std::cout << iter1 - iter2 << std::endl;        // print -1 becuase 0 - 1 = -1
std::cout << (iter1 < iter2) << std::endl;      // print 1 becuase 0 < 1 returns true
```
- iterator arithmetic may return the iterator which points to the position outside valid range, and dereferencing it is **undefined**. so you need to check before dereference it
- the result of subtracting (or adding) two iterators is like this : (position of element denoted by lhs iterator) -  (position of element denoted by rhs iterator)
    + the position start at 0
- the concept of subtracting(adding) is applied to the relational operators as well


## Arrays

### Prerequisites regarding arrays
This post assumes that you've already studied about using arrays and pointers through learning C
- hence, this post covers only about how the C++ features work with the existing concepts of using arrays in C 

### C++ features for arrays
```c++
constexpr unsigned SIZE = 42;

std::string strs[SIZE];         // ok because SIZE is constant expression; otherwise error
int& refs[10] = /* ? */         // error; there's no array for references

unsigned scores[11] = {};       // all value initialized to 0
for(auto & i : scores) {
    i = 3;                      // set all value as 3
}

auto ia2(scores);               // ia2 is unsigned int * object because the name of array is the constant pointer to the first element of an array
                                // also note that the reason why const doesn't exist is because auto ignores the top-level constness

std::string name = "kyle";
const char * str = name.c_str();    // c_str() method returns a c-style character string (array of characters and null terminated)

int int_arr[] = {0,1,2,3,4,5};
std::vector<int> iVec(std::begin(int_arr), std::end(int_arr));      // you can instantiate vector obejct by usign two iterators of the same container
                                                                    // from the first element to the last one; you can manipulate the starting and ending point
```
- you can use range for statement with arrays
- note that `c_str()` method returns pointer to constant character because `std::string` class doesn't want the array to change its value
- there are begin and end functions in a std namespace which work in a same way as as their method type does
    * `std::begin(int_arr)` returns the constexpr unsigned int * which points to the first element of int_arr



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}