---
title: "C++ Primer : Chapter 18"

categories:
    - cpp

tags:
    - [C++, Programming Language, Exception Handling, Namespace, Multiple Inheritance, Virtual Inheritance]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-21
---

# Tools for Large Programs

> 이 포스트는 C++ Primer(5th Edition)를 바탕으로 작성되었습니다.

## Exception Handling
**Exception handling** allows us to handle problems that arise at run time
- because we've covered the basics of the exception handling in the [previous chatper](https://sadoe3.github.io/cpp/primer-chapter5/#try-catch-blocks), this chapter covers the more detailed information regarding it

### Throwing an Exception
The process to search for the proper `catch` clause is known as **stack unwinding**
- local objects are automatically destroyed during stack unwinding
- the compiler uses the thrown expression to copy initialize a special object known as the **exception object**
    * if the expression has an array or function type, the expression is converted to its corresponding **pointer** type

### Catching an Excpetion
The **exception declaration** in a `catch` clause looks like a function parameter list with exactly **one** parameter
- as in a parameter list, we can **omit** the name of the `catch` parameter if the `catch` has no need to access the thrown expression
- the type can be an lvalue reference but may not be an rvalue reference
- as with a function parameter, the derived-to-base conversion is supported
    * therefore, multiple `catch` clauses with types related by inheritance must be **ordered** from most derived to least derived
- a `catch` clause can pass its current exception object out to another `catch` clause by **rethrowing** the exception
    ```c++
        try {
            try {
                throw std::exception("haha");
            }
            catch (std::exception) {
                throw;          // rethrow the current exception
            }
            catch (...) {       // catch all exception
                std::cout << "aa" << std::endl;     // this clause is skipped
            }
        }
        catch (...) {
            std::cout << "bb" << std::endl;
        }
        // prints bb
        // because rethrowing leaves current try-catch block
    ```
    * note that when you **rethrow** the exception, expression is not provided
        + because it needs to pass current exception 
    * if you want to catch all exceptions at once, you can do so by providing `...(ellipsis)` for the exception declaration in a `catch` clause
        + because it catches every exception, it must be the last `catch` clause

### Function try Blocks and Constructors
In order to handle the exception from a constructor initializer list, we must write the consturctor as a **function `try` block**
```c++
// example of writing a function try block
ClassName::ClassName(std::string inputName) try : name(inputName) { 
    /* body */ 
} catch(const std::exception &e) {
    // handle the exception
} 
```

### The noexcept Exception Specification
A function can specify that it does not throw exception by providing a `noexcept` specification
- if the function has `noexcept` specifier, then we say that function has a **nonthrowing specification**
- `noexcept` specifier must appear on **all** of the declarations and the corresponding definition of a function or on **none** of them
    * the keyword appears after the parameter list and before the trailing return
    * we may also specify `noexcept` on the declaration and definition of a function pointer
        + it may not appear in a `typdef` or type alias
    * in a member function the `noexcept` specifier follows any `const` or `reference` qualifiers
        + and it precedes `final`, `override`, or `= 0` on a virtual method
- it's possible that a `noexcept` function **does throw**; it's possible to be compiled
    * however, if it does, `terminate` is called
    * thereby enforcing the promise not to throw at run time
- note that `throw()` has the same meaning as `noexcept` although it's been deprecated in the current stardard
- the `noexcept` specifier takes an optional argument that must be convertible to `bool`
    * if the argument is `true`, then it's saying the function won't throw
    * otherwise, then it's saying the function might throw
    * the arguments to the `noexcept` **specifier** are often composed using the `noexcept` **operator**
        + the operator is an unary operator that returns a `bool` rvalue constant expression that indicates whether a given expression might throw
        + like `sizeof`, `noexcept` does not evaluate its operand
        + `void f() noexcept(g());` :
            - if `g()` doesn't throw an exception then it would be `void f() noexcept(true)`
            - otherwise, `void f() noexcept(false)`

### Exception Class Hierarchies
The `std::exception` is the root of the hierarchy
- there are 4 direct derived classes from it
    * `std::bad_cast`
    * `std::bad_alloc`
    * `std::runtime_error`
        + there are 3 direct derived classes from it
        + `std::overflow_error`
        + `std::underflow_error`
        + `std::range_error`
    * `std::logic_error`
        + there are 4 direct derived classes from it
        + `std::domain_error`
        + `std::invalid_argument`
        + `std::out_of_range`
        + `std::length_error`
- you can implement your own exception class by inheriting from the `std::exception` class or from one of the library classes derived from `std::exception`


## Namespaces
When an application uses libraries from many different vendors, it's almost inevitable that some of these names will **crash**
- if you name them very long ones, this might solve the problem, but it's not an ideal solution though
    * therefore, C++ supports **namespaces** to solve this naming collision problem in the sophisticated way 
- the following codes show the basic use of the **namespaces**
```c++
namespace custom_name {
    // namespace member
}   // doesn't need a semicolon
```
- any declaration or definition that can appear at global scope can be put into a `namespace`
    * like usual blocks, it doesn't end with a semicolon
    * it can be nested
    * it may not defined inside a function or a class
- it can be  **discontiguous**
    * which means that the definition of the namespace can appear multiple times
    * if the namespace does not refer to a previously defined namespace
        + then, a new namespace with that name is created
    * otherwise, this definition opens an existing namespace and adds the members to that already existing namespace
- there are rules of thumb regarding header and source file
    * header file
        + the definition of a class and
        + the declaration of a function or objects that are part of the class interface
        + are defined in the header file
        + and the header file is included by files that use those namespace members 
    * source file
        + the definitions of namespace members, such as methods, or functions
        + are defined in the source file
        + the definitions can be put in **separate** source files
            - if namespaces define multiple, unrelated types
            - then, they should use **separate** files to represent each type (or each collection of related types) that the namespace defines
- it's possible to define a namespace member outside its namespace definition
    * only if the member is specified by the `::(scope operator)` with the name of the namespace
    * however, template specialization must be declared in the same namespace that contains the original template
        + if it's declared inside the proper namespace, we can define it outside the namespace definition as well

### Global Namespaces
Names defined at global scope are defined inside the **global namespace**
- the **global namespace** is implicitly declared and exists in every program
- we can explicitly access to the names inside the global namespace by using `::` with an **empty** name
    * `::globalObject = 3;` is equivalent to `globalObject = 3;`

### inline Namespaces
There is a special namespace which is called `inline` namespace
- it's used for the **nested** namespace
    * if we define the nested namespace as `inline`, then
    * we can access to the members of the nested `inline` namespace as if they were the **direct** members of the enclosing namespace
- you can make the namespace as `inline` by placing the `inline` keyword before the `namespace` keyword
- the typical example of this is the codes which contain the previous version and current version
    ```c++
    // cur_version.h
    inline namespace cur_version {
        int member;
        // other members
    }

    // prev_version.h
    namespace prev_version {
        int member;
        // other members
    }

    // final_name.h
    namespace final_name {
        #include "cur_version.h"
        #include "prev_version.h"
        // other members
    }  

    // source file
    #include "final_name.h"
    ... // some codes
    final_name::member = 3;      // equivalent to final_name::cur_version::member = 3
    final_name::prev_version::member = 1;
    ```  
- note that if you include the standard header, `< >` is used
    * if you include the custom header, `" "` is used

### unnamed Namespaces
If you provide **empty** name for the namespace, then the **unnamed** namespace is created
```c++
// problem
int a = 3;
namespace {
    int a = 3;
    // members
}
a = 2;      // error; ambiguous call

// solution
namespace static_namespace {
    namespace {
    int a = 3;
    // members
    }
}
static_namespace::a = 1;        // ok

```
- the members in this namespace have `static` life time
    * which means that they are created before their first use and
    * destroyed when the program ends
- the **unnamed** namespace may be **discontiguous** within a **given** file but does **not span** files
    * if 2 files contain unnamed namespaces, those namespaces are **unrelated**
    * moreover, if a header defines an unnamed namespace, the names in that namespace define **different** entities which are **local** to each file that includes the header 
- the members in this namespace are used directly like global objects
    * if the naming collision matters, then making the namespace **nested** inside a ordinary namespace would be a good solution

### Easy Use of Namespaces
There are 3 ways to easily use the members of the namespaces
- `using` declaration
    * `using namespace_name::member;`
    * this is the feature that we've already covered
    * it introduces only **one** member of the namespace to the given scope
        + but if it's the name of the overloaded functions, it introduces all of them
    * it can appear in global, local, namespace, and class scope 
- `using` directive
    * `using namespace namespace_name;`
    * it introduces **whole** members of the namespace to the given scope
    * the target namespace must be defined before the `using` directive
    * it can appear in gloal, local, and namespace scope
        + but may not appear in a class scope
    * although this looks useful, in general, it's better to avoid `using` directives
- namespace **aliases**
    * `namespace new_name = original_name;`
    * it can be used for the nested one as well
- a header that has a `using` directive or declaration at its top-level scope **injects** names into every files that includes the header
    * as a result, headers should not contain `using` directives or declarations except inside a function or a namespace

### Overloading and Namespaces
Name lookup for names used inside a namespace follows the **normal** lookup rules
```c++
namespace A {
    int print(int);
}
namespace B {
    double print(double);
}
using namespace A;
using namespace B;
long double print(long double);
int main() {
    print(1);           // calls A::print(int)
    print(3.1);         // calls B::print(double)
    return 0;
}
```  


## Multiple and Virtual Inheritance

### Multiple Inheritance
**Multiple inheritance** is the ability to derive from more than one direct base class
```c++
class A { /* some codes */ };
class B { /* some codes */ };
class C : public A, public B { 
    /* some codes */
};
```
- each base class has an optional access specifier
    * if the specifier is omitted, the default access specifier is used as usual

### Constructors and Multiple Inhertiance
The order in which base classes are constructed depends on the order in which they appear in the class derivation list
```c++
class C : public A, public B {
    C(const C &input) : A(input), B(input), memberA(input.memberA) { }
    // other memebers
}

void ambi(const A & a);
void ambi(const B & b);
C c;
ambi(c);        // ambiguous call


struct A { 
    int member;
    /* some codes */ };
struct B { 
    int member;
    /* some codes */ };
struct C : public A, public B { 
    void f() {
        member = 3;     // ambiguous call
        A::member = 3;  // ok;
    }
    /* some codes */
};
```
- if you skip to call the base's constructor, the default constructor is called as usual
- multiple inhertiance supports **dynamic binding** for sure
    * of course, those two base classes have any relationship between each other
        + which means that they can use their interface only
    * note that converting from the derived type to each base class is **equally** good
        + which means there's chance to call ambiguously
- it's possbile for the derived class to inherit from the multiple base classes that have the members which contain the **same name**
    * this also raises the chance of ambigous call
    * using the `::(scope operator)` would be a good solution

### Copy Control
Destructors work in the same way
- they are always invoked in the reverse order from which the constructors are run
- the copy, move operations are synthesized in the same manner

### Virtual Inheritance
When you use multiple inheritance, there's chance to face the circumstance where the derived class might inherit the **same base** indirectly from two of its own direct base classes, or it might inherit a particular class directly **and** indirectly through another of its base classes
- by default, when this case happens, the derived class would contain two **duplicate** part
    * in order to solve this, C++ supports **virtual inheritance** so that the two direct base classes which inherit from the same base class **share** the it
    * therefore, the derived class now contains only single part of the duplicated base 
- you can implement the **virtual inheritance** by make the two direct base classes **virtually** inherit from the deuplicated base
    * we can do so by placing `virtual` keyword after or before the access specifier in the derivation list
        + the order is not significant
- note that the `virtual` base is initialized by the **most derived** constructor
    * this doesn't mean that the two direct base classes don't have to construct the duplicated base
        + they should construct it for sure
    * however, although they have the constructor to call their duplicated base constructor, if the most derived class is defined, then that class calls the constructor of the virtual base instead of them
        * the order of construction is same as the multiple inheritance with one exception
            - the virtual base constructor is called first
            - and the direct base constructors don't call their duplicated base's constructor if the most derived class's constructor has already called it
        * also it's worth noting that if the most derived class doesn't call the virtual base's constructor, the default constructor is called 
            - although the two direct base classes call it from their constructor 
- the following codes show the example of the **virtual inheritance**
    ```c++
    // example 1
    class A { 
    public:
        A(const std::string& input) : a(input) { std::cout << a << std::endl; }
    private:
        std::string a;
    };
    class B : public virtual A { 
    public:
        B(const std::string & input) : A(input + ": A is constructed by B"), b(input) { std::cout << b << std::endl; }
    private:
        std::string b;
    };
    class C : virtual public A { 
    public:
        C(const std::string& input) : A(input + ": A is constructed by C"), c(input) { std::cout << c << std::endl; }
    private:
        std::string c;
    };
    class D : public B, public C {
    public:
        D(const std::string& input) : A(input + ": A is constructed by D"), B(input + ": B is constructed by D"), C(input + ": C is constructed by D"), d(input) { std::cout << d << std::endl; }
    private:
        std::string d;
    };
    ...             // some codes
    D d("good");    // prints
                    // good: A is constructed by D
                    // good: B is constructed by D
                    // good: C is constructed by D
                    // good

    // example 2
    class A { 
    public:
        A() : a("") { std::cout << "A is default initialized" << std::endl; }
        A(const std::string& input) : a(input) { std::cout << a << std::endl; }
    private:
        std::string a;
    };
    // B, C have same definitions
    class D : public B, public C {
    public:
        D(const std::string& input) : B(input + ": B is constructed by D"), C(input + ": C is constructed by D"), d(input) { std::cout << d << std::endl; }
    private:
        std::string d;
    };
    ...             // some codes
    D d("good");    // prints
                    // A is default initialized
                    // good : B is constructed by D
                    // good : C is constructed by D
                    // good
    ```
- the copy control for the **virtual inheritance** works in the same way as **multiple inheritance**



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}