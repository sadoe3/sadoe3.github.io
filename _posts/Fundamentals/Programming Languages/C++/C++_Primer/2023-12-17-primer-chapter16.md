---
title: "C++ Primer : 16. Templates and Generic Programming"

categories:
    - cpp

tags:
    - [C++, Programming Language, Template]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-17
---

# Templates and Generic Programming

> 이 포스트는 C++ Primer(5th Edition)를 바탕으로 작성되었습니다.

## Defining a Template
Templates are the foundation for generic programming in C++
- a template is a **blueprint** for creating classes or functions
- we need to supply the information needed to transform that **blueprint** into a specific class or function
    * and that transformation happens during compilation

### Function Templates
A **function template** is a formula from which we can generate type-specific versions of that function
```c++
template <typename T>
inline int compare(const T &lhs, const T &rhs) {
    if(lhs < rhs) return -1;
    if(lhs > rhs) return 1;
    return 0;
}
```
- a template definition starts with the keyword `template` followed by a **template parameter list**, which is a comma-separated list of one or more **template parameters** bracketed by the < and > tokens
    * when we use a template, we specify either **implicitly** or **explicitly** the **template arguments** to bind to the **template parameters**
- the compiler uses the arguments of the call to **deduce** the template arguments
    * and the comiler uses the **deduced** template parameters to **instantiate** a specific version of the function
    * the compiler-generated functions are generally referred to as an **instantiation** of the template
- also, if you want to declare template function as `inline` or `constexpr`, you need to place the keyword **after** the template parameter list
    * otherwise, an error occurs

### Template Parameter List
Like a function parameter list, template parameters represent **types** or **values** used in the definition of a class or a function
```c++
template <class T, K> void function() {}            // error
template <class T, typename K> void function() {}   // same meaning
template <unsigned N> void function(const char (&p)[N]) {}   // same meaning
```
- they can be used as a usual type specifier
- there are two kinds of template parameters
    * template **type** parameter
        + which can be specified by the keyword `class` or `typename` which have the same meaning
        + it's used for representing the **type**
    * template **nontype** parameter
        + which can be specified by the `integral` type, or a `pointer` or (lvalue)`reference` to an object or a `function` type  
        + it's used for representing the **value**
        + the **value** is supplied by the user or deduced by the compiler
        + also the **value** must be a **constant expression**
            + so that the compiler can instantiate the template during compile time 
- each type parameter must be preceded by the proper keywords, such as `typename`, `integral` type
- template programs should try to minimize the number of requirements placed on the argument types
- we can use any name for the template parameter name
- template parameters follow the **normal scoping rules**
    * the name of a template parameter can be used after it has been declared and until the end of the template declaration or definition
    * as with any other name, a template parameter **hides** any declaration of that name in an outer scope
    * also a template parameter may not be reused within the template
    ```c++
    // error; illegal reuse of template parameter name V
    template <typename V, typename V>
    ``` 
- like a default arguments for function parameter, we can also supply **default template arguments**
    ```c++
    template <typename T = int>
    class ClassName {
    public :
        T a;
    };
    ...     // some code
    ClassName a;    // error;
    ClassName<> a;  // ok;  <> is needed although the default argument is used
    ```
    * for function templates, you can skip to type brackets(<>) to use the default arguments as usual
    * however, for class templates, you need to provide the empty bracket pair in order to use the default template arguments

### Templates and Headers
One of the features of the well-structured programs is to make appropriate use of headers
- definitions of function templates or member functions of class templates are ordinarily put into header files
- and the users of the template must include the header for the template to instantiate it properly

### Templates and Errors
In general, there are 3 stages during which the compiler might flag an error
- the first stage is when we compile the template itself
    * the compiler can detect syntax errors but not much less
- the second stage is when the compiler sees a use of the template
    * for a call to a function template, the compiler typically will check that the number of the arguments is appropriate
        + it can also detect whether two arguments that are supposed to have the same type do so
    * for a class template, the compiler can check that the right number of template arguments are provided but no much more
- the third time is when errors are detected during instantiation
    * it is only then that type-related errors can be found
    * depending on how the compiler manages instantiation, these errors may be reported at link time

### Class Templates
A **class template** is a **blueprint** for generating classes
```c++
template <typename T>
class Name {
    ... // some codes
    std::vector<T> mem;
};
```
- like function templates, class templates begin with the keyword `template` followed by a **template parameter list**
    * in the definition of the class template (and its memebers), we can use the template parameters as stand-ins for **types** or **values** that will be supplied when the template is used
- class templates differ from function templates in that the compiler **cannot** **deduce** the template parameter types for a class template
    * the template arguments must be provided to use the class templates
- each instantiation of a class template consistutes an **independent** class
    * which means, the type `Name<int>` has no relationship to, or any special access to, the members of any other `Name` type

### Member Functions of Class Templates
As with any class, we can define the methods a class template either inside or outside of the class body
```c++
// definition of method outside the class body of the template
template <typename T>
ret-type ClassName<T>::methodName(param-list) {
    ;
}


template <typename T>
class ClassName {
    ClassName methodName(param-list) {   // ok; it's inside class body
        ;
    }
};
template <typename T>
ClassName ClassName<T>::methodName(param-list) {       // error; the return type should be ClassName<T>
    ;
}
template <typename T>
ClassName<T> ClassName<T>::methodName(param-list) {       
    ClassName a;                    // ok; because it's inside class scope
    ... // some codes
}
```
- as with any other class, members defined inside the class body are implicitly `inline`
- a member function defined outside the class template body, starts with the keyword `template` followed by the class's template parameter list
    * also as usual, the name of a class generated from a template includes its template arguments
- by default, a member of an instantiated class template is instantiated **only if** the member is used
- inside the scope of a class template itself, we may use the name of the template without arguments
    * the compiler treats the name of template as if we had supplied template arguments matching the template's own parameters
        * which means, the compiler treats `ClassName` as `ClassName<T>` inside class **scope**
        * the body of the member function defined outside the class body is also treated as a class scope

### Class Templates and Friends
When a class contains a `friend` declaration, the class and the friend can **independently** be templates or not
- if the friend is a **nontemplate**, then 
    * that friend has access to all the instantiations of the template
- if the friend is a **template**, then
    * there are 3 relationships
        + one-to-one
            * this is a friendship between corresponding instantiations of a class template and its friend (class or function) 
            * in order to implement this friendship, the forward declaration of the friend is needed
            ```c++
            template <typename> class Data;
            template <typename T> class ClassName {
                friend class Data<T>;           // an instance of ClassName<T> has friendship with its corresponding Data<T>
                // some codes
            };
            ```
        + specific
            * this friendship limits the range to a specific instantiation
            ```c++
            template <typename> class Data;
            template <typename T> class ClassName {
                friend class Data<std::vector>;     // all instances of ClassName<T> has friendship with Data<std::vector>
                // some codes
            };
            ```
        + general
            * this is a friendship bewteen a certain class and every instantiation of another template
            * to implement this feature, the `friend` declaration must use template parameters that **differ** from those used by the class itself
            * also, prior declaration is not needed in this case 
            ```c++
            template <typename T> class ClassName {
                template <typename X> friend class Data;     // all instances o Data are friends of each instance of ClassName; no prior declaration needed
                // some codes
            };
            ```
- moreover, we can make a **template type parameter** a `friend`
    ```c++
    template <typename T> class ClassName {
        friend T;   // grant the access of ClassName to type T which is used to instantiate ClassName
    };
    ```
    * it is worth noting that even though a `friend` ordinarily must be a class or a function, it's okay for the class template to be instantiated with a **built-in** types

### Template Type Aliases
We can define a type alias for a class template with the following syntax
```c++
template<typename T> using newName = std::pair<T, T>;
newName<int> a;     // a is a pair<int, int>
template<typename T> using newName = std::pair<T, unsigned>;
newName<double> b;  // b is a pair<double, unsigned>
```

### static Members of Class Templates
Like any other class, a class template can declare `static` members
```c++
template <typename T> class ClassName {
private:
    static std::size_t ctr;
};
// define and initialize ClassName<T>::ctr
template <typeName T>
std::size_t ClassName<T>::ctr = 0;
```
- **each** instantiation of `ClassName` has its own instance of the `static` members
- we define a `static` data member as a template **similarly** to how we define the member functions of that template  
- we can use the `static` members of the class template in the same way as we use the `static` members of the normal class
- like any other member function of the class template, a `static` member function is instantiated only if it is used in a program

### Using Class Members That Are Types
We can use the `::(scope operator)` to access both `static` members and type members
- the class templates also support this feature
- by default, the language assumes that a name accessed through the `::(scope operator)` is **not a type** 
    * as a result, if we want to use a type member of a template type parameter, we must explicitly tell the compiler that the name is a type
    * we do so by using the keyword `typename` :
    ```c++
    template <typename T>
    typename T::value_type methodName(const T& c) {
        return c.getValue();
    }
    ```
    * note that we must use the keyword `typename` only, not `class` for this case

### Member Templates
A class (either an ordinary class or a class template) may have a member function that is itself a template
```c++
class NormalClass {
    template <typename T> void method(const T &a) { /* some codes */ }
    // other members
};
template <typename T>
class TemplateClass {
    template <typename K> void method(const K &a) { /* some codes */ }
};
```
- such members are referred as **member templates**
    * and member templates may **not** be `virtual`
- the definition of member templates is quite similar to the definition of function template 

### Controlling Instantiations
When two or more separately compiled source files use the same template with the same template arguments, there is an instantiation of that template in **each** of those files
- in large systems, the overhead of this case can become significant
    * hence, in order to avoid this overhead, we use an **explicit instantiation** like this:
    ```c++
    // in a header file
    template <typename T>
    class ClassName {
        ; // template definition
    }
    // in a certain file
    template class ClassName<std::string>;          // instantiation definition; instantiate ClassName based on the given type std::string; instantiate all members in ClassName<std::string>
    // in the other files
    extern template class ClassName<std::string>;   // instantiation declaration
    ``` 
    * when the compiler sees an `extern` template declaration, it will not generate code for that instantiation in that file
        + there may be several `extern` declaration for a given instantiation
            - the `extern` keyword declaration must appear before any code that uses that instantiation
        + but, there must be exactly **one** definition for that instantiation
            - unlike ordinary class template, the compiler instantiates **all** the members of that class although some of them may not be used
            - an instantiation definition can be used only for types that can be used with every member function of a class template
    * when you build the application, you must **link** the **object file** of the certain which handles the definition and other object files which declare only


## Template Argument Deduction
The process of determining the template arguments from function arguments is known as **template argument deduction**

### Conversions and Template Type Parameters
- there are only 2 automatic converions for template arugments
    * `const` conversions
        + as usual, top-level `const`s are ignored
        + and reference (or pointer) to a `const` can be passed a reference (or pointer) to non`const` object (low-level `const` matters)
    * array or function to pointer conversions
        + an array argument is converted to a pointer to its first element
        + a function argument is converted to a pointer to the function's type
- the codes below show the example of other case and its solutions
    ```c++
    // exmaple of a problem
    template <typename T>
    bool equalTo(const T &lhs, const T &rhs){
        return lhs==rhs;
    }
    ...         // some codes
    long lng = 3;
    equalTo(lng, 3);     // error; template parameters don't match

    // solution 1
    // use of an explicit template argument
    equalTo<int>(lng, 3);   // ok; instantiates equalTo(int, int)
    equalTo<long>(lng, 3);   // ok; instantiates equalTo(long, long)

    // solution 2
    // define two template type parameters which are compatible with each other
    template <typename A, typename B>
    bool equalTo(const A &lhs, const B &rhs){
        return lhs==rhs;
    }
    long lng = 3;
    equalTo(lng, 3);     // ok;
    ```

### Trailing Return Types and Type Transformation
Assume that you want to implement a function template that takes the itertators and returns the element from them
```c++
#include <trype_traits>

template <typename T>
auto func(T beg, T end) -> typename std::remove_reference<decltype(*beg)>::type {
    // some codes
    return *beg;
}
```
- we can obtain the type of the element by using `decltype(*beg)`
    * however, `beg` doesn't exist until the parameter list has been seen
    * to fix this issue, you must use a **trailing return type**
    * we can use it by setting the return type as `auto`, then specifying the actual return type after the parameter list with the arrow
- if you want to change the `return type`, you can do so by using **type transformation templates** which are defined in the `<type_traits>` header
    * note that because they are templates, you need to specify `typename` before using its type member
    * the table below shows the standard type transformation templates

    ||Standard Type Transformation Templates||
    |:---|:---|:---|
    |For *Mod*\<`T`>, where *Mod* is|If `T` is|Then *Mod*\<`T`>::type is|
    ||||
    |`remove_reference`|`X`& or `X` &&|`X`|
    ||otherwise|`T`|
    |`add_const`|`X`&, `const X`, or function|`T`|
    ||otherwise|`const T`|
    |`add_lvalue_reference`|`X`&|`T`|
    ||`Х`&&|`X`&|
    ||otherwise|`T`&|
    |`add_rvalue_reference`|`X`& or `X`&&|`T`|
    ||otherwise|`T`&&|
    |`remove_pointer`|`X`*|`X`|
    ||otherwise|`T`|
    |`add_pointer`|`X`& or `X`&&|`X`*|
    ||otherwise|`T`*|
    |`make_signed`|`unsigned X`|`X`|
    ||otherwise|`T`|
    |`make_unsigned`|signed type|`unsigned` T|
    ||otherwise|`T`|
    |`remove_extent`|`X`[`n`]|`X`|
    ||otherwise|`T`|
    |`remove_all_extents`|`X`[`n1`][`n2`]...|`X`|
    ||otherwise|`T`|

### Template Argument Deduction and References
- Type deduction from lvalue reference function parameters
    ```c++
    template <typename T> void f1 (T&); // argument must be an lvalue
    f1(i);          // i is an int; T is int
    f1(ci);         // i is a const int; T is const int
    f1(5);          // error; a & parameter only refers to an lvalue

    template <typename T> void f1 (const T&); // argument can be rvalue
    f1(i);          // i is an int; T is int
    f1(ci);         // i is a const int; T is int
    f1(5);          // a const & parameter can be bound to an rvalue; T is int
    ```
- Type deduction from rvalue reference function parameters
    ```c++
    template <typename T> void f1 (T&&);
    f1(3);          // argument is an rvalue of type int; T is int
    ```
- **reference collapsing**
    * there are 2 exceptions to normal binding rules which are related to how `std::move` operates
        1. when we pass an **lvalue** to a function parameter that is an rvalue reference to a template type parameter (e.g, **`T&&`**)
            * then, the compiler deduces the template type parameter as the argument's lvalue reference type
            * example: `f(3) -> T = int` but `f(i) -> t = int&`
            * with this exception, we can **indirectly** define **a reference to a reference**
                + by default, we can't do so (directly)
        2. if we indirectly create a reference to a reference, then those references **collapse**
            * for a given type `X` :
            * `X& &`, `X&& &`, and `X& &&` (rvalue reference to lvalue reference) all collapse to type `X&`
                + conceptually, you can think of the former part (`X&`) as the referenced part, and the later one (`&&`) as the referencing part
            * `X&& &&` (rvalue reference to rvalue reference) collapes to `X&&` (rvalue reference)
            * reference collapsing applies only when a reference to a reference is created indirectly, such as in a type alias or a template parameter
    * hence, we can say that the function template which takes an rvalue parameter
        + can take the rvalue
            - in this case, the final type of the parameter is `T&&`
        + also, can take the lvalue
            - in this case, the final type of the parameter is `T&` although it's defined as `T&&` in the template definition
    * therefore, rvalue reference parameters are used in one of two contexts
        + the template is **forwarding** its arguments
        + or, the template is overloaded
        + for now, it's worth noting that the function templates that use rvalue references often use overloading in the same way as we've covered before
        ```c++
        template <typename T> void f(T&&);      // binds to nonconst rvalues
        template <typename T> void f(const T&);      // binds to lvalues and const rvalues
        ```
- how `std::move` is defined
    * the standard defines `move` as follows:
    ```c++
    template <typename T>
    typename remove_reference<T>::type&& move(T&& t) {
        return static_cast<typename remove_reference<T>::type&&>(t);
    }
    ```
    * it's worth noting that we can explicitly cast an lvalue to an rvalue reference using `static_cast`
- forwarding
    * if you want to **forward** the argument while preserving its `lvalue/rvalue`ness and `const`ness to another function, you can do so by using `std::forward` which is defined in the `<utility>` header
    ```c++
    void process(int&& x) {
        std::cout << "Processing rvalue: " << x << std::endl;
    }
    void process(int& x) {
        std::cout << "Processing lvalue: " << x << std::endl;
    }
    template <typename T>
    void forwarder(T&& arg) {
        process(std::forward<T>(arg));  // Forwarding with std::forward
    }
    ... // some codes
    int a = 10;
    forwarder(a);            // Calls process(int& x) — forwards as lvalue
    forwarder(20);           // Calls process(int&& x) — forwards as rvalue
    ```
    ```c++
    // same code but without std::forward
    template <typename T>
    void forwarder(T&& arg) {
        process(arg);       
    }
    ... // some codes
    int a = 10;
    forwarder(a);            // Calls process(int& x)
    forwarder(20);           // Calls process(int& x)
    ```
    * note that if you don't use `std::forward()` the lvalue version is always called
        + this is because when you pass `20`, `arg` would be `rvalue` reference
        + but the point is that although `arg` is `rvalue` reference, it is **lvalue** inside forwarder
            - this is because the `rvalue` reference has its **own name** and **memroy address**
            - and this is why it is treated as `lvalue` when it's passed to another function although it is a `rvalue` reference
- C++ treats arguments passed to functions as `lvalue`s by **default**, regardless of whether the argument is an lvalue reference or an rvalue reference.
    * Therefore, in order to pass the argument as an `rvalue`, you should use
        + `std::move`
            - if you want to pass the argument as an **rvalue** reference **regardless** of whether it's a lvalue or rvalue
        + or `std::forward`
            - if you want to pass the argument as an lvalue **or** an rvalue based on its **original** value category
            - this is called **perfect forwarding**
- unlike `std::move`, `std::forward` must be called with an **explicit** template argument
- as with `std::move`, it's a good idea not to provide a `using` declaration for `std::forward`


## Overloading and Templates
Function templates can be overloaded by other templates or by ordinary, nontemplate functions
- as usual, functions with the same name must differ either as to the number or the type(s) of their parameters
- correctly defining a set of overloaded function templates requires a good understanding of the relationship among types and of the restricted conversions applied to arguments in template functions

### Multiple Viable Templates
The following example shows the case that both templates are viable and both provide an exact match:
- the first version: `T` is bound to `const std::string`* 
- the second version: `T` is bound to `const std::string` 
```c++
template <typename T> std::string getString(const T &t){
    return t.substr(1, 3);
}
template <typename T> std::string getString(T *t){
    return t->substr(1, 3);
}
...         // some codes
std::string s("hi");
const std::string *sp = &s;
std::cout << getString(sp) << std::endl;     // getString(T *) is selected; because it's more specialized template 
```
- when there are several overloaded templates that provide an **equally** good match for a call, the most **specialized** version is preferred
    * a function template which uses `reference` to take its paramenter is regarded as more **general**

### Nontemplate and Template Overloads
The following example shows the case that the ordinary function and function template are viable and both provide an equally good match:
- the first version: the ordinary, nontemplate function
- the second version: `T` is bound to `std::string` 
```c++
std::string getString(const std::string &s) {
    return s.substr(1, 3);
}
template <typename T> std::string getString(const T &t){
    return t.substr(1, 3);
}
...         // some codes
std::string s("hi");
std::cout << getString(s) << std::endl;     // getString(const std::string &) is selected; because it's nontemplate
```
- when a nontemplate function provides an equally good match for a call as a function template, the **nontemplate** version is **preferred**
    * because C++ regards nontemplate version as more specialized one
- declare every function in an overloaded set before you define any of the functions
    * that way you don't have to worry whether the compiler will instantiate a call before it sees the function you intended to call 


## Variadic Templates
A **variadic template** is a template function or class that can take a arbitrary number of parameters

### Defining Variadic Templates
```c++
template<typename... Args>
void f(Args ... args) {
    std::cout << sizeof...(Args) << std::endl;  // number of type parameters
    std::cout << sizeof...(args) << std::endl;  // number of function parameters
}
```
- the varing parameters are known as a **parameter pack**
    * there are 2 kinds of parameter packs
        + a **template parameter pack** represents zero or more template parameters
            - but it's worth noting that although same types are given, the size of the template parameter pack keeps increasing
        + and a **function parameter pack** represents zero or more function parameters
- we use an ellipsis (`...`) to indicate that a template or function parameter represents a pack
    * in a template parameter list
        + `typename...` or `class...` indicates that the following parameter represents a list of zero or more types
        + the name of a type followed by an ellipsis represents a list of zero or more nontype parameters of the given type
    * in the function parameter list
        + a parameter whose type is a template parameter pack is a function parameter pack
- when we need to know how many elements there are in a pack, we can use the `sizeof...` operator
    * like `sizeof` operator, `sizeof...` returns a constant expression and does not evaluate its argument

### Recursive Use and Pack Expansion
Variadic function templates are often **recursive**
```c++
template <typename T>
std::ostream& print(std::ostream &os, const T &t) {
    return os << t;
}
template <typename T, typename... Args>
std::ostream& print(std::ostream &os, const T &t, const Args&... rest) {
    os << t << " ";

    // suppose that the pack consists of a, b, c, d
    functionName(1, report(rest)...);   // it's equivalent to functionName(1, report(a), report(b), report(c), report(d));
    functionName(1, report(rest...));   // it's equivalent to functionName(1, report(a, b, c, d))
    return print(os, rest...);  // it's equivalent to return print(os, a, b, c, d);
}
```
- a declaration for the nonvariadic version of `print` must be in scope when the variadic version is defined
    * otherwise, the variadic function will recurse infintely
- in order to **expand** the function parameter pack, we need to place `...(ellipsis)` after the name of the pack
    * there are various **patterns** for expanded element
    * in general, the result of the pack expansion is like a comma-separated list which contains zero or more elements of the pack
    * the pattern in an expansion applies separately to each element in the pack
- it's also possible to **forward** the function parameter pack to another function
```c++
template <typename... Args>
void func(Args&&... args) {
    work(std::forward<Args>(args)...);
} 
```


## Template Specialization
If you want to implement a template which has a speical definition for a specific type, you can do so by using the **template specialization**

### Specializing Function Templates
```c++
template <typename T>
void func(const T& a) {
    std::cout << "default version" << std::endl;
}
template <>
void func(const int& a) {
    std::cout << "specialized version" << std::endl;
}
...         // some codes
func(1);            // instantiates the specialized version
func(1.0);          // instantiates the default version
func("haha");       // instantiates the default version
```
- to indicate that we are specializing a template, we use the keyword `template` followed by an **empty** pair of angle brackets(**<>**)
- templates and their specializations should be declared in the same header file
    * if a specialization declaration is missing, the compiler will usually generate code using the original template
    * therefore, declarations for all the templates with a given name should appear first, followed by any speicializations of those templates

### Specializing Class Templates
Like function templates, it's possible to specialize class templates
```c++
// basic use of specializing class template
template <typename T> class ClassName {
    // default definition
};
template <>
class ClassName<std::string> {
    // specialized definition
    void memberFunction();
};
void ClassName<std::string>::memberFunction() {
    // specialized definition
}



// original
template <class T> struct remove_reference {
    typedef T type;
};
// partial specialized; lvalue references
template <class T> struct remove_reference<T&> {
    typedef T type;
};
// partial specialized; rvalue references
template <class T> struct remove_reference<T&&> {
    typedef T type;
};

// original
template <typename T, typename U>
class MyClass { ; };
// Partial specialization where the second type is int
template <typename T>
class MyClass<T, int> { ; };



// specialize the member only
template <typename T> class Foo {
    void memberFunction() { /* default definition */ }
    // other members
};
template <>
void Foo<int>::memberFunction() {
    // specialized definition
}
```
- we can specify some, but not all, of the template parameters or some, but not all, aspects of the parameters
    * this is the **partial specialization**; and only class template can be partially specialized
    * users must supply arguments for those template parameters that are not completely fixed by the specialization
    * `std::remove_reference` type is the typcial example of this case 
- it's possible to specialize just the member of the class template


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}