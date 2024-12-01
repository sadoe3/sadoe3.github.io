---
title: "Predicate in C#"

categories:
    - csharp

tags:
    - [Programming Language, C#, predicate]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2021-10-28
---

## Predicate
- A Predicate is a delegate that takes either no input or a single input, and returns a `bool` value.
- Predicates are expressed using lambda expressions and are commonly used with generic collections.
    - Note that to use generic collections, you need to include the `System.Collections.Generic` namespace.
- The usage involves passing the predicate as an argument, which can be done as follows.
    ```c#
    // example of using predicate in C#
    List<string> tempList = new List<string> { "A", "B", "C" };
    Console("Find A: " + tempList.Find(x => x == "A"));
    // x is an element of the given collection object
    // the name of the element doesn't have to be x, you can name it freely like s or so -> ex: s => s == "A"
    // x == "A" is the condition
    ```
- The following is an example of using a predicate without any input in a Unity script.
    ```c#
    while(.../* some condition */) {
        ... // some codes
        yield return new WaitUntil(() => tempList.Count > 0);
        // if you don't want to pass any argument, then you need to type empty parenthesis () instead of the name of the element
    }
    ```
- The following is an example of using a predicate with a body that contains more than two lines.
    ```c#
    List<string> tempList = new List<string> { "A", "B", "C" };
    Console("Find A: " + tempList.Find(x => 
        { 
            ... // some codes
            return x == "A"
        }));
    ```

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}