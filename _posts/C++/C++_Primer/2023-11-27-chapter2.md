---
title: "C++ Primer : Chapter 2"

categories:
    - cpp

tags:
    - [C++, Programming Language, Type]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-11-27
---

# Variables and Basic Types

> 이 포스트는 C++ Primer(5th Edition)를 바탕으로 작성되었습니다.

## Primitive Built-in Types

### Arithmetic Types
The built-in types of C++ consist of 2 parts
- Arithmetic Types
    * Arithmetic Types also consist of 2 parts
    * Integral types
        + `bool, char, wchar_t, char16_t, char32_t, short, int, long, long long`
        + note that the size of `int` depends on the system
        + hence, it's better to think that it's not a fixed size
    * Floating-point types
        + `float, double, long double`
- `void` type
    * we can use `void` type in various cases
        + `void` pointer is a good example

### Signed and Unsigned Types
We can set the type as `signed` or `unsigned` except for `bool` and `extended character` types
- if it's not provided, the default is `signed`
    * however, `char` may be `signed` or `unsigned` depending on the compiler's choice
- just `unsigned` means `unsigned int`
- `unsigned` and `signed` types have the same size ,but we can use more positive part by using `unsigned` types because it extends the positvie range as long as the negative part

### Deciding Which Type to Use
- if you have confidence that negative value is not given
    * try to use `unsigned` types
- regarding integral types,
    * try to use `int` as the default choice
    * use `short` if it's too short
    * use `long long` if it's too long
        + generally, `long` has the same size as the `int`, hence it's unlikely to use `long` often
- if you use `char` in expressions, specify whether it's `unsigned` or `signed`
    * generally, `char` is used to store a single character
- regarding floating-point types
    * try to use `double` as the default choice
    * because `float` is too inaccurate
    * and `long double` is too expensive

### Type Conversions
If `Type B` is provided to where `Type A` is expected, the **automatic** (implicit) type conversion from `Type B` to `Type A` happens if the `Type B` implements `type conversion operator` to `Type A`
- basically there are type conversions between built-in types
    * `bool` types take only `0` as `false`, others are treated as `true` 
    * if you give floating-point values to integral types, the fractional part is **trunctated** (not rounded)
    * if you give integral values to floating-point types, fractional part is `0`
- if you give out-of-range value,
    * `unsigned` types would take the value as the least unsigned integer congruent to the source integer 
        + `ex: unsigned char -> 300 -> 44`
    * for signed types, the result is undefined
        + it may have the same result as `unsigned` types
- if there are `unsigned` value and `signed` value in the same expression
    * by default, `signed` value is implicitly casted to `unsigned` value
    * hence, it's usually better not to use them in the same expression

### Literals
There are various definitions for the **literals**
- I'm convinced with this defintion
    * value with name = `object`
    * value without name = `literal`
    * object which are able to change its value = `variable`
    * object which are not able to change its value = `constant`
- the general idea of a **literal** is that it's space defined by a certain type to contain the data of that type
- there are various types of **literals**
- integral literal
    * decimal = `20`
    * octal = `024`
    * hexadecimal = `0x14`
    * minmum size = `int`
- floating-point literal
    * decimal point = `3.14159`
    * scientific notation = `3.14159E0`
    * default type = `double` -> can be overriden
- string literal
    * character = `'a'`
        + type = `char`
    * C-sytle string = `"a"`
        + C-style string literal is an array which contains `char` elements
        + the last element of it is always a `\0(null character)`
        + hence `"a"` has the size of `2`
        + moreover, if there are 2 string literals that appear adjacent to one another and that are separated only by spaces, tabs, or newlines
            - they are treated as a single concatenated string literal
            - `ex: "aaa" "bbb" = "aaabbb"`
- escape sequence
    * it's used to print **nonprintable** characters
    * `\n, \t, \a, \v, \b, \", \\, \?, \', \r, \f`
- boolean literal
    * `true`, `false`
- pointer literal
    * `nullptr`
- specifying the type of a Literal
    * you can use the literal of the type of your choice not the default one
    * you can do so by providing proper suffix or prefix
        + for instance, `L'a'` uses `wchar_t`
        + `3.14L` uses `long double`
            - when you use `long double`, try to use capital letter so that it't not confused as `1`


## Variables

### Two types
C++ has 2 types 
- built-in types
- class types

### Variables Definitions
Because we haven't covered about class types, this section covers about defining object of built-in types
- the basic syntax is like this :
    ```c++
    (type specifier) (list of one or more variables, comma separated, may have initial value) ;
    
    int width, height = 3;
    ```

### Initialization
**Initialization** is a process to set the initial value to the object when it's defined
- you can do so by using `=`
    * however, if you use `=` after the object is defined, then it would be `assignment operation`
- there are 4 ways to initialize an object
    * int age = 3;
    * int age = {3};
    * int age{3};
    * int age(3);
- the ways with  `{ }`(curly braces) are called **list initializations**
    * when there's chance for data loss during **list initializations**
    * the compiler stops initializations and causes an error
    * ex: `int age{3.14}; // error`

### Default Initialization
The object is **default initialized** if you don't supply the initializer
- the default value is used to default initialize it
    * the value varies depending on the type of the object and where it's defined
- there are 2 cases for objects of built-in types
    * outside function body 
        + the default value is `0`
    * inside function body
        + in this case, the object is **uninitialized**
        + which means that the value is **undefined**
        + generally, using the **undefined** object is error-prone
            - hence, it's usually better to initialize the object when they are defined

### Variable Declaration
If you want to use the object defined in other source file, you can use it by using `extern` keyword
```c++
extern int i; // declares but does not define i
int j;        // declares and defines j

extern double pi = 3.1416 // defintion
```
- however, it's worth noting that if you use explicit initializer for the object with `extern` keyword
    * it would be the definition
- note that the definition of the object must exist in a **single** file
    * the declaration of it may exist in **multiple** files
- you cannot use `extern` keyword inside a function body

### Identifier
Identifier is a name of an object, a function, or a class.
- basically, most words can be used as identifiers except for some words which is used by the language (such as keywords)
    * but it's better to obey naming convetions, such as :
        + variable = camelCase
        + function = pascalCase
        + class = PascalCase
        + constant = SCREAMING_SNAKE_CASE
    * try to use them **consistently** no matter which naming convetion you choose

### Scope of a Name
Most scopes in C++ are delimited by curly braces
- the object defined inside a certain code block is destroyed after the control flow exits that scope
- if you want the object to be accessed from everywhere in the program, define it outside the function body
    * then it becomes a global object


## Compound Types
Compound types are built from the base type
- There are 2 sorts of compound types
    * references
    * pointers

### Reference
A reference is not an object, but an **alternative name** for an existing object
```c++
int i = 3;
int &r = i;
```
- place & before the identifier to define reference
- There's no memory allocated for the reference
- A reference must be intialized when it's created
- the initializer which is used to initialize a reference is usually an object neither literal nor expression 
    * when the object is used as the initializer, the value of it is **not copied** but the reference is bound to that object
- like constant, once the reference refers to certain object, it's not possible for that reference to refer to different object 
- the type of reference must match the type of the object which it refers to
    * except for 2 cases
        + const (type) & can refer to (type)
        + dynamic binding which will be convered in later chapter
- regarding lvalue and rvalue
    * the default reference is lvalue
    * more details would be covered in the later chapter
- you can define a reference which refers to an array
    ```c++
    int arr[10] = {};
    int (&rArr)[10] = arr;
    ```

### Pointer
A pointer is an object which contains an address of another object as its value
```c++
int i = 3;
int *p = &i;
*p = 2;
```
- place `*` before identifier to define pointer
- we can use pointers in the almost same way as we use them in C language
    * & (address-of operator)
    * \* (dereferencing)
    * double pointer
    * void pointer
- because pointers are objects, it's possible for them to contain invalid values
    * if you dereference them, an error may occur
    * therefore, it's usually better to initialize them as `nullptr`
    * there are 3 ways to do so
        ```c++
        int * p = nullptr;
        int * p = 0;
        int * p = NULL;
        ```
    * they have the equivalent meaning, but using `nullptr` is preferred
- we can define the reference which refers to a pointer
    * int* &r = p;
- you can define a pointer which points to an array
    ```c++
    int arr[10] = {};
    int (*pArr)[10] = &arr;
    ```

### Type Modifiers
Symbols, such as `*`, `&`, which are used to define pointers and references are called **type modifiers**
- we can define object with the following syntax :
    * `(base type) (list of declarators);`
    * the `declarator` consists of `(type modifier) + (identifier)`
- there are 2 ways to write **type modifiers**
    * int* p = nullptr;
    * int *p1, *p2;
- try to use one of them **consistently** no matter which way you choose

### const Qualifier
```c++
const int SIZE = 512;
```
- place const before base type
- constant object must be initialized
- constant object cannot be changed
- compilers generally convert `const` objects to literal values
- if you want to use `const` objects in multiple files
    * you must provide `extern` keyword to both of its definition and declaration 

### reference with const
Constant reference is just another name of a reference which refer to a constant object
```c++
const int VALUE = 3;
// normal(non-constant) reference
int & nRef = VALUE; // error
// constant referecne (reference to a constant object)
const int & cRef = VALUE // ok
```
- it's not possible for normal reference to be bound to `const` object
    * only, it can be bound to only non`const` object
- constant reference can be bound to every type 
    * `int value = 3;`
    * `double dVal = 3.14;`
    * `const int & r1 = value;` 
        + bound to non-constant
        + although the object which the reference refers to is non`const`, `r1` **regards** the target as `const` object,
            - hence it's not possible to change the value of the target through `r1`
    * `const int & r2 = 41;`
        + bound to literal
    * `const int & r3 = r1 * 2;`
        + bound to result of an expression
    * `const int & r4 = dVal;`
        + bound to different type
        + when it comes to `r4`, it's not bound to `dval` but the following codes are executed by compiler
            - `const int temp = dVal; const int & r4 = temp;`
            - the name `temp` is what I named arbitrarily, the actual name is not significant
            - the point is that `r4` is bound to the **temporary unnamed object** not the existing object when this circumstance happens

### pointer with const
Pointers can be used in 2 ways with `const` qualifier
```c++
int target = 3;
// top-level const : constant pointer
int * const cPtr = &target;
// low-level const : pointer to constant
const int * pCon = &target;
```
- constant pointer (top-level const)
    * place const after *
        + `int * const ptr = nullptr;`
    * the pointer itself is a constant
        + pointer must be initialized like other constant objects
        + pointer cannot change the object which it points to
    * object which pointer points to is not a constant
        + can change the value of the target object 
    * pointer can point to non-constant object **only**
- pointer to constant (low-level const)
    * place const before *
        + `const int * ptr = nullptr;` = `int const * ptr = nullptr;`
    * the pointer itself is not a constant
        + pointer need not be initialized
        + pointer can change the object which it points to
    * object which pointer points to is a constant
        + cannot change the value of the target object
    * pointer can point to constant object or non-constant object
- copy operation
    * top-level const is ignored
        + `int * const cPtr = nullptr; int * nPtr = cPtr;` OK
    * low-level const matters
        + low-level const must match when you do copying
            - if it's different, you must match them through **conversions**
        + the basic principle of conversion is like this :
            - non-const -> const : O
            - const -> non-const : X
        + `const int * pCon = nullptr; int * nPtr = pCon;` error (const -> non-const)
        + `int * nPtr = nullptr; const int * pCon = nPtr;` ok (non-const -> const)
    * there's no top-level const for references
        + because reference is itself a constant

### constexpr
Constant expression is an expression with 2 features
```c++
int num = 0;
while(num < 5)
    num++;
constexpr int cEp = num; // error: num is set at run time
constexpr int cEp = 3;   // ok
```
- it's evaluated at compile time
- the values inside expression cannot change
- examples: literals, const object which is initialized with constant expression
    + non-const object which is initialized with constant expression is not constant expression
- use **constexpr** declaration before the base type to verify whether the const object is initialized with the constant expression
    + if initializer is not constant expression, then compiler would cause an error
- literal type is the type which can be used inside constant expression
    + primitive types, compound types, enumerations, classes which are defined in a specific way (not normal classes) are the literal types
- constexpr with pointer
    + `const int * ptr = nullptr;` pointer to constant
    + `constexpr int * ptr = nullptr;` constant pointer which is constant expression
    + `constexpr const int * ptr = nullptr;` if you want to use both of constexpr and const

### Type Alias
Type Alias is another name of an existing type
```c++
// there are 2 ways to use type aliases in C++

// typedef : tradtional C style
typedef int newint;     // now newint = int
newint a = 3;           // same as int a = 3;
typedef newint A, *B;   // int = newint = A;  int* = B;
// special case for typedef
typedef char* pString;
const pString cStr;     // same as char * const cStr; not const char *;
const pString * cStr;   // same as char*const * cStr;


// using : modern C++ style
using anotherint = int; // now anotherint = int
anotherint b = 3;       // same as int b = 3;
```

### auto
we can let compiler to deduce the type of an object by using its initializer if you use auto type specifier
```c++
auto a = 3;             // same as int a = 3;
auto i = 0, *p = &i;    // ok
auto sz = 0, pi = 3.14; // error
```
- hence, an object with type of auto must be initialized
- if you define multiple objects by using auto type specifier, then those object must have the same base type
- by default, auto ignores top-level const
    + if you want to set the auto as top-level const, you need to place put const before auto
    + `const int VALUE = 3; const auto i = VALUE;`
- auto respects low-level const as usual
- also, auto ignores references
    + if you want to set auto as reference, place & after auto
    + `int a = 3; auto & b = a;`

### decltype
when you use decltype type specifier, you can set the type of an object as the return type of the given expression of decltype type specifier
```c++
decltype(f()) a;    // f() is not called, but a has whatever type f returns
decltype(1+2.3) b;  // same as double b;

const int ci = 0, &cj = ci;    // ci has top-level const, cj is reference to constant
decltype(ci) x = 0;     // same as const int x = 0;
decltype(cj) z = 3;     // same as const int & z = 3;

int i = 3, *p = &i;
decltype((i)) a = i;        // same as int & a = i;
decltype(i) b = i;          // same as int a = i;
decltype(*p) c = i          // same as int & c = i; 
```
- unlike auto, decltype handles top-level const and references
- if you use double parentheses for an object as an expression, decltype always deduce the type as the reference of the given object
- however, without double parentheses, delctype gives reference only if the given object is a reference
- decltype returns a reference type for expressions that yields objects that can stand on the left-hand side of the assignment operation


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}