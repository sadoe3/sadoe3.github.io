---
title: "Effective C++ : Chapter 4"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective C++, Class, Design]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-28
---

# Designs and Declarations

> 이 포스트는 Effective C++(3rd Edition)를 바탕으로 작성되었습니다.

## Item 18

### Make interfaces easy to use correctly and hard to use incorrectly
When you design a class, try to make interfaces easy to use correctly and hard to use incorrectly so that you can get **help from the compiler**
```c++
// premature design
class Data {
public:
    Data(int month, int day, int year);
};

// refined design
struct Day {
    explicit Day(int d) : val(d) {}
    int val;
};
struct Month {
    explicit Month(int m) : val(m) {}
    int val;
};
struct Year {
    explicit Year(int y) : val(y) {}
    int val;
};

class Data {
public:
    Data(const Month& month, const Day& day, const Year& year);
};
Date(30, 3, 1995);                    // error; wrong types!
Date(Day(30), Month(3), Year(1992));  // error; wrong types!
Date(Month(3), Day(30), Year(1992));  // ok; types are correct
```
- instead of using only `int`s, using `enum`s might be better
- try to use `const` whenever posssible to avoid unnecessary value change
- ways to facilitate correct use include consistency in interfaces and behavioral compatibility with built-in types
- ways to prevent errors include creating new types, restricting operations on types, constraining object values, and eliminateing client resource management responsibilities


## Item 20

### Prefer pass-by-reference-to-`const` to pass-by-value
If the parameter of the function doesn't need to be changed, 
- try to define it as reference-to-`const`
    * if the type of it is **user-defined**
    * with this approach, you can avoid unnecessary copy operation, possible slicing problem, and prevent the value from being changed
- try to define it as local object
    * if the type of it is **built-in** or **STL iterator** or **STL callable object** types
    * this approach is usually preferred because C++ compiler typically implement references as pointers
        + hence passing a reference is actually passing a pointer
        + which means that if the target type is built-in type
        , then using the actual type can be more efficient
        + same rules are applied to STL iterator or STL callable object types


## Item 21

### Don't try to return a reference when you must return an object
When you return a reference, always make sure that the reference returned refers to the object that exists at that time
```c++
// example of the bad code
const ClassName operator*(const ClassName &lhs, const ClassName &lhs) {
    ClassName result(lhs.a * rhs.a, lhs.b * rhs.b);
    return result;              // result would be destroyed after exiting this function
}
```


## Item 22

### Declare data members `private`
Declaring data members `private` gives clients syntactically uniform access to data, affords fine-grained access control, allows invariants to be enforeced, and offers class authors implmentation flexibility
- `protected` data members are no more encapsulated than `public`
    * because the **encapsulatedness** of a data member is **inversely proportional** to the **amount of code** that might be broken if that data members **changes**


## Item 23

### Prefer non-member non-friend functions to member functions
As the above item explains, the **number of functions** that interact with the `private` data members is **inversely proportional** to the **encapsulatedness**
- the point is that the member function which calls such functions is also **counted** 
    * hence, if you want to define a function which calls such functions, try to define it as non-member and non-friend
- moreover, define different functions in different headers although they are in the same `namespace`
    ```c++
    // header webbrowser.h
    // header for class WebBrowser itself
    // as well as core WebBrowser-related functionality
    namespace WebBrowserStuff {
        class WebBrowser { /* definition */ };
    }

    // header webbrowserbookmarkss.h
    namespace WebBrowserStuff {
        ...                     // bookmark-related convenience functions
    }

    // header webbrowsercookies.h
    namespace WebBrowserStuff {
        ...                     // cookie-related convenience functions
    }
    ```


## Item 24

### Declare non-member functions when type conversions should apply to all parameters
When you define an **overloaded operator**, if there's chance where the **left-hand side object** is constructed through a conversion from another type
- then define it as non-member function
    ```c++
    // problem
    class Rational {
    public:
        Rational(int numerator = 0, int denominator = 1);
        const Rational operator*(const Rational& rhs) const;
        // other members
    };
    Rational oneEighth(1,8);
    Rational oneHalf(1,2);

    Rational result = oneHalf * 2;      // fine
    Rational result = 2 * oneHalf;      // error!
    
    // solution 
    class Rational {
        // same definition without operator*
    };
    const Rational operator*(const Rational& rhs) {
        return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
    }
    Rational result = oneHalf * 2;      // fine
    Rational result = 2 * oneHalf;      // fine!
    ```


## Item 25

### Consider support for a non-throwing `swap`
The standard library's typical implementation is exactly what you'd expect:
```c++
namespace std {
    template <typename T>
    void swap(T& a, T& b) {
        T temp(a);
        a = b;
        b = temp;
    }
}
```
- note that there 3 copy operations in the default implementation
    * if this is not efficient for your case, then you need to implement your custom swap function
        + the typical example is using **Pimpl** (pointer to implementation) Idiom
- you can do so by following these steps:
    1. implement `public noexcept` swap member function which does the **efficient swap**
        ```c++
        class ClassName {
        public:
            void swap(ClassName& other) {
                using std::swap;            // necessary
                swap(pImpl, other.pImpl);   // do the efficient swap
            }
        };
        ```
    * 2-A: if the target is the normal class
        ```c++
        namespace std {
            template<>
            void swap<ClassName>(ClassName& a, ClasName& b) {
                a.swap(b);                  // call the custom version of swap
            }
        }
        ```
        + specialize `std::swap`, and make it call your custom `public noexcept` swap method
    * 2-B: if the target is the class template
        ```c++
        namespace MyNamespace {
            template <typename T>
            class ClassName { /* definition */ };   // includes custom swap method
            // other members

            // non-member swap function
            // not part of the std namespace
            template <typename T>
            void swap(ClassName<T>& a, ClassName<T>& b) {       
                a.swap(b);
            }
        }
        ```
        + define non-member swap function in the same namespace as your class template, and make it call your custom `public noexcept` swap
- the reason why there 2 different cases for step 2 is because it's fine to totally specialize `std` templates for user-defiend types,
    * but never try to add something completely new to `std`
    * and to partially specialize function template means adding a completely new overloaded one
        + hence we can't use `std` namespace for 2-B case
- when calling `swap`, employ `using` declaration for `std::swap`, then call `swap` without namespace qualification
    * otherwise, the `swap` function defined in 2-B case would never be called becaue you need to call `std::swap`

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}