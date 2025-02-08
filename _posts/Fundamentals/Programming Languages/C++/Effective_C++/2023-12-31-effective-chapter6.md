---
title: "Effective C++ : Chapter 6"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective C++, Inheritance, OOP]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2023-12-31
---

# Inheritance and Object-Oriented Design

> 이 포스트는 Effective C++(3rd Edition)를 바탕으로 작성되었습니다.

## Item 32

### Make sure public inheritance models "is-a"
Public inheritance means "is-a"
- everything that applies to base classes also apply to derived classes
    * because every derived class object **is a** base class object


## Item 35

### Consider alternatives to virtual functions
There are 4 alternatives to normal `virtual` functions
```c++
// normal virtual function
class GameCharacter {
public:
    virtual int getHealthValue() const;
    // other members;
};

// NVI
class GameCharacter {
public:
    int getHealthValue() const {
        // some codes
        int curHealth = getHealthReal();       // do the real work
        // some codes
        return curHealth;
    }
    // other members;
protected:      // this should be protected not private
    virtual int getHealthReal() const {
        // some codes
    }
};

// Strategy Pattern: Classic Use
class GameCharacter;
class HealthCalcFunc {
public:
    // some codes
    virtual int calc(const GameCharacter& gc) const { /* some codes */ }
    // some codes
};
HealthCalcFunc defeaultHealthCalc;
class GameCharacter {
public:
    explicit GameCharacter(HealthCalcFunc *phcf = &defeaultHealthCalc) 
    : pHealthCalc(phcf) {}
    int getHealthValue const { return pHealthCalc->calc(*this); }
    // some codes
private:
    HealthCalcFunc* pHealthCalc;
};

// Strategy Pattern: via function pointer
class GameCharacter;
int defeaultHealthCalc(const GameCharacter& gc);
class GameCharacter {
public:
    using HealthCalcFunc = int(*)(const GameCharacter&);
    explicit GameCharacter(HealthCalcFunc phcf = defeaultHealthCalc) : pHealthCalc(phcf) { }
    int getHealthValue() const { return pHealthCalc(*this); }
private:
    HealthCalcFunc pHealthCalc;
};

// Strategy Pattern: via std:function
class GameCharacter;
int defeaultHealthCalc(const GameCharacter& gc);
class GameCharacter {
public:
    using HealthCalcFunc = std::function<int (const GameCharacter&)>;
    explicit GameCharacter(HealthCalcFunc phcf = defeaultHealthCalc) : pHealthCalc(phcf) { }
    int getHealthValue() const { return pHealthCalc(*this); }
private:
    HealthCalcFunc pHealthCalc;
};
```
- Non-Virtual Interface (NVI) idiom 
- Strategy Pattern
    * classic use
    * via function pointer
    * via `std::function`


## Item 38

### Model "has-a" or "is-implemented-in-terms-of" through composition
Unlike public inheritance, you can choose whether
- providing all operations or
- provoding only some of them through different interfaces if needed
- thorugh **composition**


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}