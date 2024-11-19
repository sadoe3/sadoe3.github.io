---
title: "Clean Code : Chapter 9"

categories:
    - coding-practice

tags:
    - [Clean Code, Coding Practice, Test, Unit Test]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-9
---

# Unit Tests

> 이 포스트는 Clean Code(1st Edition)를 바탕으로 작성되었습니다.

## Tests
A production code supports various **functionalities**
- and **each functionality** should be **tested before** being part of the production code

### Unit Tests
**Unit tests** are executed to test those **functionalites**
- and the **unit tests** must be **clean** as well
    * this is because tests **matter**
    * systems that are **not testable** are **not verifiable**
        + arguably, a system that **cannot be verified** should **never be deployed**

### Clean Tests
- a **clean test** includes
    * good **readability**
    * single `assert()`
    * the fact that it tests **single concept** only
- moreoever, a **clean test** also follow the 5 other rules which form the acronym (*F.I.R.S.T*)
    * **FAST** : Tests should be fast. They should run quickly. When tests run slow, you won't want to run them frequently. If you don't run them frequently, you won't find problems early enough to fix them easily. You won't feel as free to clean up the code. Eventually the code will begin to rot.
    * **Independent** : Tests should not depend on each other. One test should not set up the conditions for the next test. You should be able to run each test independently and run the tests in any order you like. When tests depend on each other, then the first one to fail causes a cascade of downstream failures, making diagnosis difficult and hiding downstream defects.
    * **Repeatable** : Tests should be repeatable in any environment. You should be able to run the tests in the production environment, in the QA environment, and on your laptop while riding home on the train without a network. If your tests aren't repeatable in any environment, then you'll always have an excuse for why they fail. You'll also find yourself unable to run the tests when the environment isn't available.
    * **Self-Validating** : The tests should have a boolean output. Either they pass or fail. You should not have to read through a log file to tell whether the tests pass. You should not have to manually compare two different text files to see whether the tests pass. If the tests aren't self-validating, then failure can become subjective and running the tests can require a long manual evaluation
    * **Timely** : The tests need to be written in a timely fashion. Unit tests should be written just before the production code that makes them pass. If you write tests after the production code, then you may find the production code to be hard to test. You may decide that some production code is too hard to test. You may not design the production code to be testable.


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}