---
title: "Clean Code : Chapter 3"

categories:
    - coding-practice

tags:
    - [Clean Code, Coding Practice, Function]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-8
---

# Functions

> 이 포스트는 Clean Code(1st Edition)를 바탕으로 작성되었습니다.

## Single Purpose

### One Function One Purpose
It's important to implement **a function** which achieves **one purpose**
- if it achieves more than one
    * then, you need to **decompose** it
- the point of this mechanism is to define the **level of abstraction** regarding the purpose
    * simply, there might be 3 levels :
        + **low-level** of abstraction
            - the function of this level would execute code like
                * `text.append("\n");`
        + **intermediate-level** of abstraction
            - the function of this level would execute code like
                * `String pagePathName = PathParser.render(pagePath);`
            - **intermediate-level** purpose can be achieved through achieving one or more **low-level** purposes
        + **high-level** of abstraction
            - the function of this level would execute code like
                * `getHtml();`
            - **high-level** purpose can be achieved through achieving one or more **intermediate-level** purposes
- moreover, it's preferred for programmers to write functions based on the level of abstraction
    * so that we can read the program, descending one level of abstraction at a time as we read down the list of functions
    * for instance
        + *in order to achieve `A`, we need `A-A` and `A-B`*
        + *in order to achieve `A-A`, we need `A-A-1` and `A-A-2`*
        * *in order to achieve `A-A-1`, ...*
    * the author call this *The Stepdown Rule*

### Don't Repeat Yourself
Write code once, use it often
- if you see multiple lines of code which do the same thing exist in various places, define a single function which contains them, and replace them with the call to that function
    * if you do so, it would increase the **maintainability**
        + because the change in one place would affect the whole places where it's called
- there are various pinciples and practices which are used to eliminate the code duplications
- it's worth noting that **code duplication** may happen because of the **improper decomposition** of the function


## Function Parameters
The **ideal number** of **parameters** for a function is **zero**.
- next comes one, followed closely by two
    * **three** arguments should be **avoided** where possible
    * **more than three** requires very speical **justification**
        + when a fuction seems to need more than two or three arugments, it is likely that some of those arguments ought to be **wrapped** in to a class
            - [**aggregate classes**](https://sadoe3.github.io/cpp/primer-chapter7/#aggregate-classes) can be useful for this case
        + or 


## Exception Handling
Prefer **exceptions** to returning error codes
```c++
// bad
if(deletePage(page) == E_OK) {
    if(registry.deleteReference(page.name) == E_OK) {
        if(configKeys.deleteKey(page.name.makeKey()) == E_OK) {
            logger.log("page deleted");
        } else {
            logger.log("configKey not deleted");
        }
    } else {
        logger.log("deleteReference from registry failed");
    }
} else {
    logger.log("delete failed");
    return E_ERROR;
}
// possibly more code


// clean
try {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
} catch(std::exception e) {
    logger.log(e.what());
}
// no more code except for possible return statement
```
- when you return an **error code**, you create a problem that the caller must deal with the error immediately
    * on the other hand, if you use **exceptions**, then the error processing code can be **separated** from the main processing code and can be **simplified**
- moreover, if the function supports the exception handling, then
    * the whole main processing code should be located inside the `try` block
        + so that the first word in the function is `try`
    * and there sould be **nothing** after the `catch` blocks for C++(`catch/finally` blocks for Java)
        + except for a **`return` statement** which returns default object or something like that as the return value for the `catch` blocks 


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}