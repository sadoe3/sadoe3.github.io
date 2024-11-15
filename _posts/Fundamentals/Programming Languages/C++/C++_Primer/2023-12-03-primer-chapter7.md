---
title: "C++ Primer : 7. Classes"

categories:
    - cpp

tags:
    - [C++, Programming Language, Class]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-3
---

# Classes

> 이 포스트는 C++ Primer(5th Edition)를 바탕으로 작성되었습니다.

## Defining Abstract Data Types

### 3 Core Concepts of OOP
There are 3 core concepts of Object-Oriented Programming.
- Data Abstraction
    * it is the ability to define both data members and member functions so that the class **abtractly** implement a certain concept
    * you can think of the implmentation as the whole definition of the class, and the interface as the members of the class which the users of the class can execute
- Encapsulation
    * it is to **separate** the interface and the implementation so that the users of the class can use the interface but have no access to the implementation
- Dynamic Binding
    * this concept is related to **inheritance** which would be covered in the later chapter

### Abstract Data Type
A class that uses **data abstraction** and **encapsulation** defines an **abstract data type**.
- in an abstract data type, only the class designer worries about how the class is implemented
- programmers who use the class need not know how the type(class) works
    * they can instead think **abstractly** about what the type does

### 2 Types of members
The class has 2 types of its members.
```c++
// exmaple of the defintion of the class
struct ClassName {
    int dataMember;

    double memberFunctionA() { return 3.2; }
    double memberFunctionB();
};
... // some codes
ClassName a;        // suppose the default-intialization is implemented
a.memberfunctionA();
```
- data member
    * data members must be declared inside the class body
    * data members may be initialized inside the class body or through the constructor
- member function (method)
    * member functions msut be declared inside the class body
    * member functions may be defined inside or outside the class body
- class definition consists of the keyword `class` or `struct` and the name of the class and the body of the class
- the user of the class can access to its member by using `.(dot operator)`
- you must put `semi-colon` after the ending braces of the class definition

### this parameter
Every member function of the class has **implicit** `this` parameter.
```c++
struct ClassName {
    int dataMember;
    double memberFunctionB() {
        return this->datamember * 3.22;
    }
};
```
- `this` parameter is a **constant pointer** which points to the object of the class on which the member function was invoked
- because `this` parameter is **reserved** for this special use, we can't set the name of the paramter as "this"
    - also, because `this` is the **implicit** parameter, it's not visible from the paramter list
- any direct use of a member of the class is asssumed to be an implicit reference through `this`

### const member function
Because `this` is the **implicit** parameter, there's no way to set it as the constant pointer to a constant by using the exisiting concept.
- in order to implement this, we need to learn the new concept : **const member function**
```c++
struct ClassName {
    const int a = 3;
    int memberFunction() const {
        return a*3;
    }
    int memberFunction() {
        return a;
    }
};
...     // some codes
ClassName a;
a.memberFunction(); // 3 is returned
const ClassName b;
b.memberFunction(); // 9 is returned
```
- when we place `const` after the parameter list, we set the member function as the **const member function**
    * it makes `this` as the constant pointer to a constant
    * so that you can't change the value of the data member through this member function unless they are `mutable`
- it's possible to overload member functions based on the `const`
    * note that the **non-const object** can call both of **const member functions** and **non-const member functions**
        + because implicit conversion from non-const to const is valid
    * whereas, the **const object** can call **const member functions** only

### Order of compilation
When compiler compiles the class, it compiles the declarations of members first, and then, it compiles the functions bodies of the member functions only after the entire class has been seen
```c++
struct ClassName {
    int memberFunction() {
        return a;       // ok;
    }
    const int a = 3;
};
``` 
- although some data members are defined after the definition of the member function in the class body, the function still can use them
    * because they're compiled first

### Defining a method outside the function body
When you define a method outside the function body, just do the same thing as you define a normal function with one exception : `ClassName::methodName()`
```c++
struct ClassName {
    int memberFunction();   // method declaration
    const int a = 3;        // in-class initialization
};
int ClassName::memberFunction() {       // defining the method outside the class body
    return a;
}
```
- you need to use `::(scope operator)` in order to access to the member function becuase the member function is in the class(scope)

### return *this
It's possible to return the object itself by returning `*this`
```c++
struct ClassName {
    ClassName& memberFunction() { return *this; };
    const ClassName& memberFunction() const { return *this; };
};
```
- because **const member function** has `this` as the constant pointer to the constant, the return type of `*this` should be the reference to the constant

## Specialized Member Functions
There are 3 types of the specialized member functions
- constructors
    * a constructor is a member function which is called first **automatically** when the object of the class is created
    * the job of the constructor is to **initialize** its data members and to execute certain codes if needed right after the initialization phase 
- overloaded operators
    * you can overload various operations on the classes by implementing overloaded operations
- destructors
    * a destructor is a member function which is called **automatically** when the object of the class is destroyed
- the overloaded operators and destructors would be covered in the later chapter

### Synthesized Member Functions
When you don't implement the specialized member functions, compiler creates them instead
- they are called as **synthesized member functions**
- constructor
    * if you don't declare any constructors, compiler creates the **default constructor**
    * **default constructor** initializes the data members as follows
        + if there is an in-class initializer, use it to initialize the member
        + otherwise, default-initialize the member
- overloaded operators
    * copy, and assign operators are made if they are not implemented
- desturctors
    * if you don't declare the destructor, it's created by the compiler as well

### Constructors
Constructors have the following features
- they don't have the return type
- they have the same name as the class
- they can be overloaded with different parameters
    ```c++
    struct ClassName {
        ClassName() = default;   // default constructor
        ClassName(const std::string & inputName, const int & inputWeight) : name(inputName), weight(inputWeight) {}
        ClassName(const int & inputWeight);

        int age = 3;             // in-class initialization
        std::string name;
        double weight;
    };

    ClassName::ClassName(const int & inputWeight) : name("nobody"), weight(inputWeight) {}
    ```
    * you can overload normal member functions as well
- if there are constructors declared in the class definition, compiler doesn't create the default constructor
    * in this case, we can **request** the compiler to implement the default constructor by using `= default` to the constructor with empty parameter
- note that we initialize the data member inside **Constructor Initializer List** where is after parameter list and before function body
    * we initialize the data members by using direct initialization
    * the only job of the function body is to contain some codes which need to be executed right after the initialization phase
    * if you omit some data members inside the **Constructor Initializer List**, they would be default-initialized unless there're in-class initializers
    * therefore, it's possible not to specify **Constructor Initializer List** so that the constructor initialize the data member in the same way as default constructor does
- it's possible to define the constructor outside the class defintion
    * same rules are applied to constructor as defining methods outside with two exceptions
        + it doesn't have the return type
        + it initializes data members inside **Constructor Initializer List**
- a constant or a reference data member must be **initialized** through the in-class initializer or the **constructor initializer list** 

### Order of the Intialization inside constructor
The data members are initialized in the order in which they appear in the class definition
```c++
class ClassName() {
    ClassName(int value) : b(value), a(b) {}       // a is initialized first with the undefined value of b
    int a;
    int b;
}
```
- the important point to note is that the order **doesn't change** even if we change the order of the **constructor initializer list**
    * therefore, it's a good idea to write constructor initializer list in the **same order** as the members are declared

### Default Constructor
As we've covered already, a default contructor is a constructor which contains **no parameters**
```c++
class ClassName() {
    ClassName(int value = 3) : a(value) {}       // considered as a default constructor
    int a;
}
... // some codes
ClassName a();                    // note that this is not an object creation but a function declaration

ClassName a;                      // object is created by the default contructor
ClassName a = ClassName();        // object is created by the default contructor
```
- however, there's one more type of default constructor
    * if all parameters of the contructor have their default value, then this constructor is also considered as a default contructor
- also, you need to be aware that calling a default contructor doesn't need any **parentheses** when you direct-initialize it
    * if you do so, then it is a **function declaration**

### Delegating Constructor
A **delegating constructor** is a contructor which uses another constructor from its **own** class to perfrom its initialization
```c++
class MyClass {
    public:
        MyClass() : MyClass(3) {                    // delegating constructor
            std::cout << "1 " << value << " ";
        }
        MyClass(int input) : value(input) {         // delegated-to contructor
            std::cout << "2 " << value << " ";
        }
    private:
        int value;
};
// print result: 2 3 1 3
```
- the contructor initializer list of the delegating constructor takes **only one entry** which is the contructor call
    * you can choose which constructor to call by giving different parameters
- the order of the execution is like this:
    1. delgating constructor calls the delgated-to constructor
    2. constructor initializer list of the delegated-to constructor is executed
    3. function body of the delegated-to constructor is executed
        * it's usually empty
    4. function body of the delegating constructor

### Implicit Class Type Converions
A **converting constructor** is a contructor which takes only one parameter of the different type of class
```c++
struct ClassA() {
    ClassA(std::string inputName) : name(inputName) {}
    std::string name;
};
struct ClassB() {
    explicit ClassB(std::string inputName) : name(inputName) {}         // prevent implicit type conversion
    std::string name;
};

void funcA(ClassA a) {
    std::cout << a.name;
};
void funcB(ClassB a) {
    std::cout << a.name;
};
...     // some codes
funcA("kyle");                              // error; can't execute multiple implicit type conversion
funcA(std::string("kyle"));                 // ok; the temporary object is created by calling the converting constructor; this object would be discarded after the end of func()

funcB(std::string("kyle"));                 // error; you need to call the constructor explicitly
funcB(ClassB(std::string("kyle")));         // ok;
```
- when you pass the object of the different to the position where a certain type is expected, it's legal if there's a converting constructor between those 2 types
    * then, the implicit type conversion happens
    * however, note that it's impossible to execute multiple implicit type conversions
        + in this example : `c-style string` -> `std::string` -> `ClassA`
        + in order to achieve multiple type converions, you need to perform explicit type conversions so that there's only one conversion left which can be done in implicit way
- if you want to prevent the use of a constructor in a context that requires an implicit conversion, you can achieve this by delcaring the converting constructor as `explicit`
    - the **explicit converting constructor** is called only if it's called explicitly

## Access Specifiers
There are 3 types of **access specifiers** which control the access level of the members
- `public`
    * members defined after a `public` specifier are accessible to all parts of the program
    * hence, the `public` members define the **interface** to the class
- `protected`
    * it's related to the concept of **inheritance**, hence it would be covered in the later chapter
- `private`
    * members defined after a `private` specifier are accessible to the member functions of the class but are **not** accessible to the code that uses the class
    * hence, the `private` sections **encapsulate(hide)** the implementation

### How to specify the access level
```c++
class ClassName {
public:
    ClassName() = default;   // default constructor
    ClassName(const std::string & inputName, const int & inputWeight) : name(inputName), weight(inputWeight) {}

private:
    int age = 3;             // in-class initialization
    std::string name;
    double weight;
};
... // some codes
ClassName a;
a.age;                      // error; impossible to access private member
```
- you can put the public section after `public:` and the private section after `private:`
- if there's no access specifier provided inside the class defintion, then the **default access level** is used to specify the access level of the members
    * `class` has the `private` as its **default access level**
    * `struct` has the `public` as its **default access level**
        + this is the **only difference** between `class` and `struct` in C++

### Non-member Class-Related Functions
- in some cases, you need to implement functions which handle the class, but is not part of the class
    * these functions should be declared in the same header as the class itself

### Friend Functions
If you want the non-member class-related functions to have the access to the implementation of the class, you can set it by declaring the function as the **friend** of the class
```c++
class ClassName {
friend int func(ClassName a);
public:
    ClassName() = default;   // default constructor
    ClassName(const std::string & inputName, const int & inputWeight) : name(inputName), weight(inputWeight) {}

private:
    int age = 3;             // in-class initialization
    std::string name;
    double weight;
};
int func(ClassName a);       // declaration or definition should exist before the call of the function
```
- note that, by default, the `friend` declaration is just to give the access to the function which means it is not the general function declaration
    * hence, you need to declare or define the function before the call of it
- also, the access level is not important for `friend` declaration because it doesn't mean anything to the function  

## Additional Class Features

### Type Members
A class can define its own local names for types
```c++
class ClassName {
public:
    typedef std::string::size_type s_pos;
    using v_pos = std::vector::size_type;
private:
    s_pos cursor = 0;
    v_pos width = 3;
};
```
- it can be implemented in both ways : `typedef` and `using`
- you can think of type members as the data members
    * hence, you need to specify the access level
    * also, you need to define type members before declaring the data members which use them as their type
        + as a result, type members usually appear at the beginning of the class

### inline member functions
You can set the member functions as `inline`
- if you define the member functions inside the class body, they are **implicitly** `inline` member functions
- you can **explicitly** define the member function as `inline` by placing `inline` before return type like normal function
    * this can be done inside the class body or outside as well

### mutable data members
If you want to change the value of the data member inside the **const member function**, you can achieve this by setting the data member as `mutable`
```c++
class ClassName {
public:
    int memberFunction() const { 
        age = 1;             // error;
        weight = 3.1;        // ok
    }

private:
    int age = 3;             // in-class initialization
    std::string name;
    mutable double weight;
};
```
- you can set the data member as `mutable` by placing `mutable` keyword before the type of the member

### in-class initialization of the data member of the class type
You can initialize the data member of the class type inside the class body
```c++
class ClassName {
    std::string name = std::string(3, 'c');         // ok
    std::string name(3, 'c');                       // error
    std::string name{3, 'c'};                       // ok
};
```
- you can achieve this in 2 ways
    * `= constructor()`
    * `{ }`
        + note that you need to use curly braces to do the direct initialization not just normal parentheses

### Class Declaration
You can declare the classes
```c++
class ClassName;

ClassName funcA(ClassName &, int);  // ok
int funcA(double a, ClassName b) {  // error
    ... // some codes
};

int main() {
    ClassName objectName;       // error
    ClassName * ptr;            // ok
}

class Class2 {
    Class2* next;               // ok
    Class2* prev;               // ok
};
```
- This declaration is sometimes called **forward delcaration**
- the class which has the declaration but not the definition is the **incomplete type**
    * the incomplete type is utilized in only limited ways
        + it's possible to define compound types to the incomplete types
        + it's possible to declare (**not define**) functions that use the incomplete type as a parameter or return type
            - therefore, it's possible for a class to have a pointer to the type of itself as its data member
    + otherwise, we are unable to use it
        + it's not possible to create the object of the incomplete type
        + it's not possible for the compound types to the incomplete types to access the object of the incomplete types 

### Friend Classes
It's also possible to make another **class** or the **member function** of another class as a `friend` of the class
```c++
class ClassB;               // need to be declared first so that ClassA can have the parameter of ClassB

class ClassA {
public:
    int member(ClassB input);   // need to be defined after the definition of ClassB because it's incomplete type at this point
};

class ClassB {
friend class ClassA;                        // you can set the class as its friend
friend int ClassA::member(ClassB);          // or the member function as its friend

private:
    int age = 3;
};

int ClassA::member(ClassB input) {
    return input.age;
}
...              // some codes
ClassA a;
ClassB b;
a.member(b);            // ok
```
- when you make the class or a member function as a friend, the **order** of declaration and definition matters
    1. you need to declare the `ClassB` first in order to support incomplete type feature for `ClassA`
    2. then, the member function of `ClassA` is declared but not defined yet until the `ClassB` is defined
    3. after `ClassB` is defined, define the member function of `ClassA`


### Name Lookup regarding member functions
Generally, it's usually bad idea to use identical identifiers, however, the following exmaple is good codes to see how the name lookup works
```c++
int age = 3;        // global object
class ClassName {
    int memberFunction(int age) {   // local object
        age = 3;                // local object is used
        ClassName::age = 1;     // data member is used
        this->age = 1;          // data member is used
        ::age = 2;              // global object is used
    }

    int age;        // data member of the class
}
```
- the precedence between the 3 types of objects is like this:
    1. local object (parameter of a method, object created inside a method)
    2. data member of the class
    3. global object

### Aggregate Classes
An **aggregate class** which statifies 4 conditions has special initialization syntax
```c++
class ClassA {
public:
    int a;
    std::string b;
    int getNumber() {
        return a * 3;
    }
};
...     // some codes
ClassA a = {1, "haha"};     // aggregate class can be initialized with a braced list
```
- 4 conditions of an aggregate class
    * all of its data members are `public`
    * it does not define any constructors
    * it has no in-class initializers
    * it has no base classes or `virtual` functions which are related to the concept of inheritance
- if it statifies the conditions above
    * you can initilaize each data member of it by using **list initialiation**
- an aggregate class may have methods
- like an array, if the list of intializers has fewer elements, the trailing members are **value-initialized**
- like any constructors, the order of initializers must be same as the order of the data members

### Literal Classes
If you want to use a certain class in the **constant expression**, you can do so by making it as a **literal class**
```c++
class Debug {
public:
    constexpr Debug(bool b = true) : hw(b), io(b), other(b) { }
    constexpr Debug(bool h, bool i, bool o) : hw(h), io(i), other(o) { }
    constexpr bool any() { return hw || io || other; }
    void set_io(bool b) {io = b};
private:
    bool hw;
    bool io;
    bool other;
}
```
A class can be a literal type if it satisfies the following conditions:
- if it's an **aggregate class**, it's literal class if all of its data members are literal types
- otherwise, it must meet the following restrictions:
    * the data members all must have literal type
    * the class must have at least one `constexpr` constructor
    * if a data member has an in-class initializer for a member of built-in type must be a constant expression, or if the member has class type, the initializer must use the member's own `constexpr` constructor
    * the class must use default definition for its destructor
- literal classes may have non-`constexpr` methods
- although constructors can't be `const`, constructors in a literal class can be `constexpr`
    * actually, there must be at least on `constexpr` constructor
- it's possible to set the default constructor as `constexpr`
- a `constexpr` constructor must initialize every data member
    * and the initializer must either use a `constexpr` constructor or be a constant expression

### static members
The members of the class can be `static` if you put `static` keyword before their declarations
```c++
class ClassA {
public:
    static void initInterest() { interest = 0.6; }
    static void doSomething();      // declare only

    ClassA(double inputMoney) : money(inputMoney) {}
    void calculateMoney() { money += money * interest; } 
private:
    static double interest;
    double money;
}

void ClassA::doSomething() {        // defining outside; same as normal methods
    ...     // some codes
}
...         // some codes
ClassA::initInterest();
ClassA a;
a.calculateMoney();
```
- if the member is declared as `static`, it exists **outside** any object of the class
    * there's only **one** static member in the program, and all objects share it
        + for data members, objects **don't contain** data associated with `static` data members
        ```c++
        struct ClassA {
            public:
                int a;
                static constexpr int b = 3;
        };
        ... // some codes
        ClassA b;
	    std::cout << sizeof(b) << " " << b.b << std::endl;
        // print result: 4 3
        ```
        + but note that it's possible for objects to access `static` data member in the same way as they do so for normal data member
        + for member functions, `static` member functions are not bound to any object
            - therefore, `static` member functions don't have a `this` pointer in either of explicit way and implicit way
            - you can't access to the normal members of the class inside the `static` method
            - you can't set `static` member function as `const` because there's no `this` pointer
- you can use `static` members in 2 ways
    * `ClassName::data` or `ClassName::method()` : you don't have to create an object using this way
        + member functions of the class can use the `static` members directly, without the `::(scope operator)`
    * `object.data` or `object.method()` : you can access to the `static` members through the same way as accessing to the normal members
- generally, `static` data members are initialized through `static` methods, however, it's possible to use in-class initializer to initialize `static` data members if the following conditions are satisfied
    * the `static` data member must be `const (integral type)` or `constexpr (literal type)`
    * the initializer must be a constant expression
- there are 2 special uses of `static` data members unlike normal data members
    * `static` data members can have incomplete types
        + in particular, they can have the same type as the class type of which it is a member
        ```c++
        class Bar {
            static Bar a;           // ok
            Bar * b;                // ok
            Bar c;                  // error; normal data members must have complete type
        }
        ```
        + they can be used as the default argument of the methods
        ```c++
        class Bar {
        public:
            void doSomething(char = bg);
        private:
            static const char bg;
        }
        ```

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}