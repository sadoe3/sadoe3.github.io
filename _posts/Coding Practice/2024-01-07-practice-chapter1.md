---
title: "Clean Code : Chapter 1"

categories:
    - coding-practice

tags:
    - [Clean Code, Coding Practice, Bad Code]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-7
---

# Clean Code

> 이 포스트는 Clean Code(1st Edition)를 바탕으로 작성되었습니다.

## Leaving the Bad Code Alone
If we leave the **bad code** alone, then as we added **more and more** features, the code got **worse and worse** until we simply **cannot** manage it any longer

### Solution
The code has to be kept **clean** over time
- if we all checked-in out code a litte **cleaner** than when we checked it out
    * the code simply **could not rot**
        + otherwise, code would rot and degrade as time passes 
- the **cleanup** doesn't have to be something big
    * it could be
        + changing one variable name for the better, or
        + breaking up one function that's a little too large, or
        + eliminating one small bit of duplication or something like this

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}