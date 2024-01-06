---
title: "Effective Modern C++ : Chapter 6"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective Modern C++, Lambda Expression]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-6
---

# Lambda Expressions

> 이 포스트는 Effective Modern C++(1st Edition)를 바탕으로 작성되었습니다.

## Item 31

### Avoid default capture modes
As we've covered through [**Primer**](https://sadoe3.github.io/cpp/primer-chapter10/#lambda-expressions)
- we can capture every entity from the enclosing function by using default capture modes
- a **default by-reference capture** causes a closure to contain a reference to a **local** object or to a parameter that's available in the scope where the the lambda is defined
    ```c++
    std::vector<std::function<bool(int)>> filters;
    
    // problem
    void addDivisorFilter() {
        auto divisor = 3;

        // reference to divior will dangle right after control flow exits addDivisorFilter
        filters.emplace_back([&](int value) {return value & divisor == 0;});
    }

    // possible solution
    void addDivisorFilter() {
        auto divisor = 3;

        // it's safe because the value of divisor is copied
        filters.emplace_back([=](int value) {return value & divisor == 0;});
    }
    ```
    * if the lifetime of a closure created from that lambda **exceeds** the lifetime of the local variable or parameter
        + the reference in the closure will **dangle** 
- however, the **default by-value capture** is not perfectly safe to the dangling issue
    ```c++
    class ClassName {
    public:
        void addFilter() const;
    private:
        int divisor;
    };

    // problem
    void ClassName::addFilter() const {
        auto curObjectPtr = this;
        // the viability of this filter is tied to the lifetime of this ClassName object
        filters.emplace_back([=](int value) {return value & curObjectPtr->divisor == 0;});
    }

    void doSomeWork() {
        auto ptr = std::make_unique<ClassName>();

        ptr->addFilter();       // the filter is tied to local variable ptr
    }   // now ptr is destoyed; the filter holds the dangling poitner
    

    // possible solution
    void ClassName::addFilter() const {
        // C++14;
        // copy data member to closure
        filters.emplace_back([divisor = divisor](int value) {return value & divisor == 0;});
    }

    void doSomeWork() {
        auto ptr = std::make_unique<ClassName>();

        ptr->addFilter();       // the filter is independent to ptr; because it's copied
    }
    ```
    * note that the solution uses new capture feature covered in [**Item 32**](https://sadoe3.github.io/cpp/modern-chapter6/#item-32)


## Item 32

### Use init capture to move objects into closures
```c++
std::vector<std::function<void(void)>> filters;

class ClassName {
public:
    ClassName(const int &inputValue) : value(inputValue) {}
    void addFilter() {
        filters.emplace_back([newValue = value + 1]() {std::cout << "value: " << newValue << std::endl; });
    }

private:
    int value;
};

// some codes
{
    ClassName one(1);
    one.addFilter();
}   // one is destroyed

for (auto curElement : filters) {
    curElement();       // prints 2; it works because the value is copied
}
```
- **init capture** can do virtually everything the C++11 capture forms can do plus more
    * the one thing that you can't express with an **int capture** is a default capture mode 
    * **init capture** is often called **generalized lambda capture**
        + because you can capture the result of an expression
- to the **left** of the `=` is the name of object which is used inside the closure
    * and to the **right** is the expression which initializes that object
    * the point is that we can use the **name of the data member** inside the expression
    * moreover, we can perform `std::move` on the local object inside the expression
        + this is preferred when moving gives more efficiency than copying


## Item 33
### Use `decItype` on auto&& parameters to `std::forward` them
O

## Item 34
### Prefer lambdas to `std::bind`
In C++ only, `std::bind` may be useful for implementing move capture or for binding objects with templatized function call operators
- however, by default, lambdas are more readable, more expensive, and may be more efficient than using `std::bind`


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}