---
title: "C : Include Guard"

categories:
    - c

tags:
    - [C, C++, Include Guard, Multiple File Handling, macro, preprocessing]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-17
---

# Include Guard


## Problem
When you write a program, your `source` files often need to use various `header` files to declare functions, variables, constants, etc
- these `header` files might also **include other** `headers`
    * and in some cases, **a header** file might be **included multiple times** by different parts of the code
- if a header is included multiple times in the same file, it can lead to **problems** like:
    * **redefinition errors**
        + if a function or a class (in `C++`) is defined multiple times, the **compiler** will throw an `error`
    * **increased compilation time**
        + re-including the same headers multiple times increases the work done by the compiler

### Example
```c
// header.h
int func1() {
	return 1;
}
```
```c
// header2.h
#include "header.h"
// some codes
```
```c
// main.c
#include "Header.h"
#include "Header1.h"

int main() {
	func1();
	return 0;
}
```
- this example occurs the compilation `error`


## Solution
You can easily solve this problem by using the technique called **include guard** so that a header is **included only once**, even if the code is written to include it m**ultiple times**

### Example
```c
// header.h
#ifndef MYHEADER_H
#define MYHEADER_H

int func1() {
	return 1;
}

#endif
// other files are same
```
- now the project is executed **properly**

### Modern Alternative: `#pragma once`
Another alternative to traditional include guards is the `#pragma once` directive
```c++
// header.h
#pragma once

int func1() {
    return 1;
}

// other files are same
```
- still, the project is executed **properly**
- it tells the preprocessor to include the file only once
    * regardless of how many times it’s encountered
- it's a more **modern** and **simpler** approach, but it’s not part of the C standard (though it's supported by most modern compilers)
    * hence if you want to achieve the **portabilty** to the legacy code, try to use the **traditional way**


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}