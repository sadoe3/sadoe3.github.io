---
title: "Effective Modern C++ : Chapter 8"

categories:
    - cpp

tags:
    - [C++, Programming Language, Effective Modern C++, pass by value, emplace]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-6
---

# Tweaks

> 이 포스트는 Effective Modern C++(1st Edition)를 바탕으로 작성되었습니다.

## Item 41

### Consider pass by value for copyable parameters that are cheap to move and always copied
For copyable, cheap-to-move parameters that are always copied
- **pass-by-value** may be nearly as efficient as **pass-by-reference**
    * it's easier to implement
    * and it can generate **less object code**
    * however, **pass-by-vaule** is subject to the **slicing problem**
        + hence, it's typically inappropriate for base class parameter types
- for **lvalue** arguments,
    * **pass-by-value** (i.e., copy construction) followed by **move** assignment may be significantly **more expensive** than
        + **pass-by-reference** followed by **copy** assignment 


## Item 42

### Consider emplacement instead of insertion
In principle, **emplacement** functions should sometimes be more efficient than their insertion counter parts, and they should **never be less efficient**
- in practice, they’re most likely to be **faster** when 
	* the value being added is constructed into the container, not assigned; or
	* the argument type(s) passed differ from the type held by the container; or
	* the container won’t reject the value being added due to it being a duplicate
- emplacement functions may perform type conversions that would be rejected by insertion functions


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}