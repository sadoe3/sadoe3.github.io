---
title: "C++ Primer : 14. Overloaded Operations and Conversions"

categories:
    - cpp

tags:
    - [C++, Programming Language, Overloaded Operators, Conversions]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-12
---

# Overloaded Operations and Conversions

> 이 포스트는 C++ Primer(5th Edition)를 바탕으로 작성되었습니다.

## Basic Concepts

### Overloaded Operators
The basic concept of **overloaded operators** is covered in the [previous chapter](https://sadoe3.github.io/cpp/chapter13/)
- except for the overloaded `function-call opeartor`, an overloaded operator may not have default arguments
- if an operator function is a **member function**, the left-hand operand is bound to the implicit `this` pointer
    * in this case, the member operator functions have one less (explicit) parameter than the number of operands
- an operator function must either be a member of a class or have at least one parameter of class type
    * this restriction means that we **cannot change** the meaning of an operator for the `built-in` types
- we can overload most, but not all, of the operators
    * the following table shows the detailed information

    |Operators|
    |:---:|
    |**Operators That May Be Overloaded**|
    |+ &nbsp;&nbsp;&nbsp; - &nbsp;&nbsp;&nbsp; * &nbsp;&nbsp;&nbsp; / &nbsp;&nbsp;&nbsp; % &nbsp;&nbsp;&nbsp; ^ &nbsp;&nbsp;&nbsp; & &nbsp;&nbsp;&nbsp; \| &nbsp;&nbsp;&nbsp; ~ &nbsp;&nbsp;&nbsp; ! &nbsp;&nbsp;&nbsp; , &nbsp;&nbsp;&nbsp; = &nbsp;&nbsp;&nbsp; < &nbsp;&nbsp;&nbsp; > &nbsp;&nbsp;&nbsp; <= &nbsp;&nbsp;&nbsp; >= &nbsp;&nbsp;&nbsp; ++ &nbsp;&nbsp;&nbsp; -- &nbsp;&nbsp;&nbsp; << &nbsp;&nbsp;&nbsp; >> &nbsp;&nbsp;&nbsp; == &nbsp;&nbsp;&nbsp; != &nbsp;&nbsp;&nbsp; && &nbsp;&nbsp;&nbsp; \|\| &nbsp;&nbsp;&nbsp; += &nbsp;&nbsp;&nbsp; -= &nbsp;&nbsp;&nbsp; /= &nbsp;&nbsp;&nbsp; %= &nbsp;&nbsp;&nbsp; ^= &nbsp;&nbsp;&nbsp; &= &nbsp;&nbsp;&nbsp; \|= &nbsp;&nbsp;&nbsp; *= &nbsp;&nbsp;&nbsp; <<= &nbsp;&nbsp;&nbsp; >>= &nbsp;&nbsp;&nbsp; [] &nbsp;&nbsp;&nbsp; () &nbsp;&nbsp;&nbsp; -> &nbsp;&nbsp;&nbsp; ->\* &nbsp;&nbsp;&nbsp; new &nbsp;&nbsp;&nbsp; new [] &nbsp;&nbsp;&nbsp; delete &nbsp;&nbsp;&nbsp; delete [] |
    |**Operators That Cannot Be Overloaded**|
    |:: &nbsp;&nbsp;&nbsp; .* &nbsp;&nbsp;&nbsp; . &nbsp;&nbsp;&nbsp; ?:|

    * 4 symbols (+,-,* and &) serve as both unary and binary operators
        + the number of parameters determines which operator is beind defined
    * ordinarily, the comma, address-of, logical AND, and logical OR operators **should not** be overloaded
    * moreover, we **cannot** invent new operator symbols
- an overloaded operator has the same **precedence** and **associativity** as the corresponding built-in operator
- although we usually call an overloaed operator function **indirectly** by using the operator on arguments of the appropriate type
    * it's possible to **explicitly** call the overloaded operators
        ```c++
        // equivalent call
        data + data2;
        operator+(data, data2);
        
        // equivalent call
        data += data2;
        data.operator+=(data2);
        ```
- if you decide to define overloaded operators, use definitions that are **consistent** with the built-in meaning
    * which means, if there's a default version, such as `assignment operator`, it should have the same `parameter list` and `return type`

### Choosing Member or Nonmember Implementation
When we define an overloaded operator, we must decide whether to make the operator a class member or an ordinary nonmember function
- the following guidelines help you to decide
    * the `assignment(=)`, `subscript([])`, `function call(())`, and `member access arrow(->) operators` **must** be defined as **members**
    * the compound-assignment operators ordinarily **ought** to be **members**. however, unlike assignment, they are not required to be members
    * operators that change the state of their object or that are closely tied to thier given type-such as increment, decrement, and dereference-usually should be **members**
    * symmetric operators-those that might **convert** either operand, such as the arithmetic, equality, relational, and bitwise operators-usually should be defined as ordinary **nonmember** functions
    * IO operators must be **nonmember** function
        + because if these operators are members of any class, they would have to be members of `istream` or `ostream`
        + however, those classes are part of the standard library, and we **cannot** add members to the class in the library

## Input and Output Operators

### Overloading the Output Operator <<
```c++
std::ostream& operator<<(std::ostream &os, const Data &data) {
    os << data.dataA << " " << data.dataB << " " << data.getDataC();
    return os;
}
```
- ordinarily, the first parameter of an output operator is a reference to a non`const` `ostream` object
    * it's `reference` because we cannot copy an `ostream` object
    * `ostream` object is non`const` because writing to the stream changes its state
- the second parameter ordinarily should be a reference to `const` of the class type we want to print
    * it's `reference` because we want to avoid copying
    * it's reference to `const` because we don't want to change the value
- generally, output operators should print the contents of the object, with **minimal formatting** which means that they should not print a newline

### Overloading the Input Operator >>
```c++
std::istream& operator>>(std::istream &is, Data &data) {
    double price;
    is >> data.dataA >> data.dataB >> price;
    if(is)
        data.dataC = price * data.dataB;
    else
        data = Data();      // reset to the default object

    return is;
}
```
- ordinarily, the first parameters of an input operator is a reference to a non`const` `istream` object
- the second parameter ordinarily should be a reference to non`const` of the class type we want to print
    * it's `reference` because we want to avoid copying
    * it's reference to non`const` because we want to change the value
- generally, input operators must deal with the possibility that the input might **fail**
    * checking the `istream` object as a condition is a good way to handle error recovery

## Arithmetic and Relational Operators

### Overloading Arithmetic Operators
```c++
Data operator+(const Data &lhs, const Data &rhs) {
    Data sum = lhs;
    sum += rhs;
    return sum;
}
```
- ordinarily, arithmetic operators take 2 reference to `const` parameters as its left-hand side and right-hand side operands
- also, it returns a copy of the local object which contains the result
- classes that define both an arithmetic operator and the related compound assignment operators ordinarily **outght** to implement the arithmetic operators by using the corresponding compound assignment operators

### Overloading Equality Operators
```c++
bool operator==(const Data &lhs, const Data &rhs) {
    return lhs.dataA == rhs.dataA && lhs.dataB == rhs.dataB && lhs.dataC == rhs.dataC;
}

bool operator!=(const Data &lhs, const Data &rhs) {
    return !(lhs == rhs);
}
```
- ordinarily, eqaulity and not equality operators take 2 reference to `const` parameters as its left-hand side and right-hand side operands
- also, it returns a boolean value as a result
- classes for which there is a logical meaning for equality normally should define `operator==`

### Overloading Relational Operators
```c++
bool operator<(const Data &lhs, const Data &rhs) {
    return lhs.dataK < rhs.dataK;
}
```
- ordinarily, eqaulity and not equality operators take 2 reference to `const` parameters as its left-hand side and right-hand side operands
- also, it returns a boolean value as a result
- if a single logical definition for < exists, classes usually should define the < operator
    * however, if the class also has ==, define < only if the definitions of < and == yield **consistent** results

## Assignment Operators
This is covered in the [previous chapter](https://sadoe3.github.io/cpp/primer-chapter13/#copy-assignment-operator)
- the additional point is that we can get the class type object of the different type as the right-hand side operand
- compound-assignment operators have the same function prototype and definition with two excpetion
    * the operator for the name of the method is changed to compound assignment operator 
    * the operator is changed to compound assignment operator inside the definition


## Subscript Operators

### Overloading Subscript Operators
```c++
class Data {
public:
    std::string& operator[](const std::size_t &n) { return elements[n]; }
    const std::string& operator[](const std::size_t &n) const { return elements[n]; }
private:
    std::string *elements;
}
```
- ordinarily, subscript operators take one parameter which is a reference to `const` object which takes the index information
- if a class has a subscript operator, it usually should define **two** versions
    * one that returns a plain reference and
    * the other that is a `const` member and returns a reference to `const`


## Increment and Decrement Operators

### Overloading Prefix Increment and Decrement Operators
```c++
Data& Data::operator++() {
    check(curIndex);        // reset to 0 if curIndex is already at the last element
    ++curIndex;
    return *this;
}
Data& Data::operator--() {
    --curIndex;
    check(curIndex);        // reset to 0 if curIndex is past the first element 
    return *this;
}
```
- classes that define increment or decrement operators **should** define both the prefix and postfix verions
- ordinarily, prefix increment and decrement operators take **empty** parameter
    * and returns a reference to the incremented or decremeted object
- when we increment or decrement an object, we usually need to do the range test so that it's safe to perform the subscript operation based on the current index

### Overloading Postfix Increment and Decrement Operators
```c++
Data Data::operator++(int) {       // unused int parameter
    Data old = *this;
    ++(*this);
    return old;
}
Data Data::operator--(int) {       // unused int parameter
    Data old = *this;
    --(*this);
    return old;
}
// calling the postfix operators explicitly
Data a;
a.operator++(0);        // postfix
a.operator++();         // prefix
```
- to distinguish the prefix and postfix operators, the postfix operators take an extra **(unused)** parameter of type `int`
    * because it's not used, we do not give it a name
- ordinarily, postfix increment and decrement operators return the copy of the old object(not incremented or decremented yet)
- we call the prefix operators inside postfix operators so that prefix would do the range test instead


## Member Access Operators

### Overloading Dereference and Arrow Operators
```c++
struct Data {
    int a;
    int b;
};
class ClassA {
public:
    ClassA() : data(nullptr) {}
    Data& operator*() { return *data; }
    // delegate the real work to the dereference operator
    Data* operator->() { return &(this->operator*()); }     
    ~ClassA() {
        if (data != nullptr)
            delete data;
    }
    void setData() { data = new Data(); }
private:
    Data* data;
};
...         // some codes
ClassA a;
a.setData();
std::cout << a->a << std::endl;         // print 0
```
- operator **arrow** must be a **member**
    * the **dereference** operator is not required to be a member but usually should be a **member** as well
- note that if you don't want to change the state of the object through these operators
    * you need to define these operators as `const` member function
- the dereference operator take empty parameter and returns the reference of the object which the pointer points to
    * the type of the object or the pointer can differ based on the concept of the class
- the arrow operator take empty parameter as well and returns the pointer which points to the object which is returned from the dereference operator
    * the overloaded arrow operator must return either a pointer to a class type or an object of a class type that defines its own arrow operator
    * hence, you can think of the order of execution for arrow operator as the following
        1. `object->memberA`
        2. the part (`object->`) is change to (`pointer->`) or (`newObject->`) based on whether the overloaded arrow operator for `object` returns the a pointer to a class type or an object of a class type
            - a. if it's changed to (`pointer->`), the step continues to 3
            - b. if it's changed to (`newObject->`), it goes back to step 2
                * this iteration keeps going until the overloaded arrow operator returns a pointer to a class type
                * if the object doesn't have the overloaded arrow operator, then error occurs
        3. the built-in arrow operator is executed to access memberA
            * `pointer->memberA` is equivalent to `(*pointer).memberA` 


## Function-Call Operators

### Overloading Function-Call Operators
```c++
class ClassA {
public:
    unsigned operator() (const int & value) { return value < 0 ? -value : value;}
};
...     // some codes
ClassA a;
std::cout << a(-3) << std::endl;        // prints 3
```
- the function-call operator must be a member function
- call operator takes empty parameter, instead, it has another parameter list which follows the usual parameter list
    * if the another parameter list is different, there can be multiple overloaded call operators in the same class
- the `return type` and `additional paramemter list` is defined in the same way as we define normal function
- objects of classes that define a call operator are referred to as **function object** because they act like a function

### Lambdas Are Function Objects
When we write a `lambda expression`, the compiler translates that expression into an **unnamed** object of an **unnamed** class that contains the overloaded function-call operator
```c++
std::string sz = "some texts";
auto wc = std::find_if(words.begin(), words.end(), [sz] (const string &a) {return a.size() < sz.size();})
// the above code is equivalent to the below code
std::string sz = "some texts";
class Unnamed {
public:
    Unnamed(std::string a) : sz(a) { }      // parameter for each captured object
    // call operator with the same return type, parameters, and body as the lambda
    bool operator() (const string &a) {return a.size() < sz.size();})
}
auto wc = std::find_if(words.begin(), words.end(), Unnamed(sz));

```

### Library-Defined Function Objects
In the `<functional>` header, there are class templates which define the function-call operator that applies the named operation

||Library Function Objects||
|:---:|:---:|:---:|
|**Arithmetic**|**Relational**|**Logical**|
|`plus<Type>`|`equal_to<Type>`|`logical_and<Type>`|
|`minus<Type>`|`not_equal_to<Type>`|`logical_or<Type>`|
|`multiplies<Type>`|`greater<Type>`|`logical_not<Type>`|
|`divides<Type>`|`greater_equal<Type>`||
|`modulus<Type>`|`less<Type>`||
|`negate<Type>`|`less_equal<Type>`||

- the function-object classes that represent operators are often used to override the default operator used by an algorithm
    * `sort(svec.begin(), sev.end(), greater<string>())`

### Callable Objects and function
```c++
#include <functional>
#include <map>
// example of simple desk calculator
int add(int i, int j) { return i + j; }         // ordinary function
auto mod = [] (int i, int j) { return i % j;};  // lambda
struct divide {                                 // function-object class
    int operator() (int deno, int div) {
        return deno/div;
    }
};
// std::function version
std::map<std::string, std::function<int(int, int)>>  binops = {
    {"+", add},                                     // function pointer
    {"-", std::minus<int>()},                       // library function object
    {"/", divide()},                                // user-defined function object
    {"*", [] (int i, int j) {return i*j;}},         // unnamed lambda
    {"%", mod}                                      // named lambda object
};
binops["+"](10, 5);         // calls add(10, 5)
binops["-"](10, 5);         // uses the call operator of the minus<int> object
binops["/"](10, 5);         // uses the call operator of the divide object
binops["*"](10, 5);         // calls the lambda function object
binops["%"](10, 5);         // calls the lambda function object


// function pointer version
std::map<std::string, int (*) (int, int)> binops = {
    {"+", add},                                     // ok
    {"/", divide()}                                 // error; divide() returns the object of the class type, not pointer to function
}
```
- C++ has several kinds of callable objects:
    * functions, pointers to function, lambdas, objects created by `std::bind`, and classes that overload the function-call operator
- **call signature** specifies the `return type` and the `parameter list` of the callable objects
    * the point is that the callable objects with **same call signature** may have the **different type**
    * to solve this problem, the library defines `std::function` template that takes the callable objects with **same call signature** given through template argument and treat them as they have the **same type**
        * it's defined in the `<functional>` header
- the following table shows the operations on `std::function`

||Operations on `std::function`|
|:---:|:---:|
|`function<T> f;`|`f` is a null `std::function` object that can store callable objects with a call signature that is equivalent to the function type `T` ( i.e., `T` is `returnType(args)` )|
|`function<T> f(nullptr);`|explicitly construct a null `std::function`|
|`function<T> f(obj);`|stores a copy of the callable object `obj` in `f`|
|`f`|use `f` as a condition; `true` if `f` holds a callable object; `false` otherwise|
|`f(args)`|calls the object in `f` passing `args`|
||**Types defined as members of `std::function<T>`**|
|`result_type`|the type returned by this `function` type's callable object|
|`argument_type`|one of the types defined when `T` has exactly one or two arguments. if `T` has one argument, `argument_type` is a synonym for that type|
|`first_argument_type`|one of the types defined when `T` has exactly one or two arguments. if `T` has two argument, `first_argument_type` and |
|`second_argument_type`|`second_argument_type` are synonyms for those argument types|


## Overloading, Conversions, and Operators
A conversion between a class and another class can happen through **converting constructor** and **conversion operators**
- they define **class-type conversions**; and such conversions are also referred to as **user-defined conversions**

### Converting Constructors and Conversion Operators
```c++
// basic syntax of conversion operator
operator type() const;
// example of basic use
class SmallInt {
public:
    SmallInt(int i = 0) : val(i) {}         // converting constructor
    operator int() const {return val;}      // conversion operator
private:
    std::size_t val;
};
...                 // some codes
SmallInt si;
si = 3;                  // implicitly converts 3 to SmallInt by calling converting constructor then calls SmallInt::operator= which is synthesized by the compiler
int result = si + 7;     // implicitly converts si to int by calling conversion operator, then integer addition occurs, then initialize result with the result of the addition
std::cout << result << std::endl;        // prints 10
```
- a **converting constructor** must have one parameter which has the type different from the its own class type
- a **conversion operator** must be a **member function**, may **not** specify a **return type**, must have an **empty parameter**, and usually should be `const`
    * although it doesn't specify the `return type`, each conversion function must return a value of its corresponding type
        + if the function returns the type which is different from the type before empty `parameter list`, an error occurs
    * if there is no single one-to-one mapping between the current class type and the another type, it's better not to define the conversion operator
        * instead, the class ought to define one or more ordinary members to extract the information in these various forms

### explicit Conversion Operators
```c++
// similar class with one excpetion : explicit conversion operator
class SmallInt {
public:
    SmallInt(int i = 0) : val(i) {}         
    explicit operator int() const {return val;}      // explicit conversion operator
private:
    std::size_t val;
};
...                 // some codes
SmallInt si;
si = 3;             // ok
int a = si + 7;     // error; implicit conversion is prohibited
int a = static_cast<int>(si) + 7;   // ok
```
- you can define conversion operators of most types except for one type : `bool`
    * becaues `bool` is an arithmetic type, the implicit conversion to `bool` can be used in any context where an arithmetic type is expected
- in order to prevent such problems, we can define conversion operators as **explicit** by placing `explict` keyword before `operator`
    * if the operator is `explicit`, then we must cast the object **explicitly** with one exception :
        + the `explicit` conversion operator is called **implicitly** when the conversion to an expression is used as a condition
    * therefore, we should define conversion operators to `bool` as `explicit`
        + because they are usually intended for use in conditions
        + and if they used in conditions, they are called **implicitly** although they are defined as `explicit`
        + for other uses, `explicit` casting is the expected use for converting to `bool`

### Avoid Ambiguous Conversions
There are 2 ways that **ambiguous conversions** can occur
```c++
// example of mutal conversion
struct B;
struct A {
    A() = default;
    A(const B &);           // convert B to A
    // other members
}
struct B {
    operator A() const;     // convert B to A
    // other members
}
A f(const A&);       // function takes a parameter which have type of A
...         // some codes
B b;
A a = f(b);                 // error; ambiguous call: f(B::operator(A())) or f(A::A(const B&))
A a = f(b.operator A());    // ok;
A a = f(A(b));              // ok;


// exmaple A of the second case
struct A {
    A(int = 0);
    A(double);

    operator int() const;
    operator double() const;
    // other members
}
void f(long double);
...         // some codes
A a;
f(a);       // error; ambiguous call: f(A::operator int()) or f(A::operator double())
long lg;
A a2(lg);   // error; ambiguous call: A::A(int) or A::A(double)
short s = 42;
A a3(s);    // ok; promoting short to int is better than converting short to double

// exmaple B of the second case
struct A {
    A(int);
    // other members
}
struct B {
    B(int);
    // other members
}
struct C {
    C(double);
    // other members
}
void f(const C &);
void f(const D &);
...          // some codes
f(10);       // error; ambiguous call: f(A(10)) or f(B(10)) or f(C(double(10)))
f(A(10));    // ok

// exmaple C of the second case
A operator+(const A &lhs, const A &rhs) {
    // some codes
}
class A {
    friend A operator+(const A&, const A&);
public:
    A(int = 0);
    operator int() const {return value;}
private:
    std::size_t value;
}
...                 // some codes
A s1, s2;
A s3 = s1 + s2;     // ok; uses overloaded operator+
int i = s3 + 0;     // error; ambiguous call : s3.operator int() + 0 or int(operator+(s3, A(0)))
```
- the first happens when two classes provide **mutual conversions**
    * **mutual conversions** exist when a class `A` defines a converting constructor that takes an object of class `B` and `B` itself defines a conversion operator to type `A`
    * ordinarily, it's a bad idea to define classes with **mutual conversions**
- the second happens when the conversion-related classes have **user-defined conversions** to the same type
    * if there are multiple conversions when processing function matching, the function of best match is called, otherwise, it's the **ambiguous call**
    * ordinarily, it's a bad idea to define conversions to or from two arithmetic types
        + if you want to implement this kind of conversion, implment only one conversion to the arithmetic type
        + and let the standard conversions convert to the other arithmetic types
        + moreover, do not create mulitiple classes which can be **implicitly** constructed from the same arithmetic value
    * the set of candidate functions for an operator used in an expression can contain both nonmember and member functions
        + the `example C` covers this problem

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}