---
title: "Clean Code : Chapter 2"

categories:
    - coding-practice

tags:
    - [Clean Code, Coding Practice, Name, Naming]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-8
---

# Meaningful Names

> 이 포스트는 Clean Code(1st Edition)를 바탕으로 작성되었습니다.

## Use Intention-Revealing Names
If a name requires a **comment**, then the name does **not reveal** its intent
```c++
// bad
int d;
objectName.getThem();

// clean 
int daysSinceCreation;
objectName.getFlaggedCells();
```

### Make Meaning Distinction
Avoid number-series naming
```c++
// bad
void copyCharacters(char a1[], char a2[]);

// clean
void copyCharacters(char source[], char destination[]);
```
- because they are **noninformative**
    * which means they provide **no clue** to the author's intention


## Use Disinformation
Programmers should avoid words whose entrenched meanings **vary** from our intended meaning 

### Avoid Abbreviation
```c++
// bad
TypeName to;

// clean
TypeName temporaryObject; 
```

### Avoid Single-Letter Variable
```c++
// bad
for(int i = 0; i < MAX; i++)
    // do something

// clean
for(int currentCount = 0; currentCount < MAX; currentCount++)
    // do something
```

### Avoid Well-Known Type in a Name Unless the Object Has that Type
```c++
// bad
std::vector<Account> accountList;

// clean
std::vector<Account> accounts;
// or
std::list<Account> accountList;
```

### Avoid Lower-Case L or Uppercase O
```c++
// bad
int l = 1;
int O1 = 01;
```

### Pick One Word Per Concept
Pick one **word** for one abstract **concept** and **consistently** use it
- however, it's important to **avoid** using the **same word** for **multiple** purposes


## Use Pronounceable Names
```c++
// bad
Date genymdhms;     // which stands for generation date, yaer, month, day, hour, minute, and second

// clean
Date generationTimestamp;
```


## For Modern Systems
In days of old, when programmers worked in **name-length-challenged** languages, they violated these rules
- however, with **modern systems**, there's no need to follow
    * **Hungarian Notation**
        ```c++
        // bad
        int nSize;

        // clean
        constexpr int sizeOfArray = 3; 
        ```
    * **Member Prefixes**
        ```c++
        // bad
        class ClassName {
            std::string m_dsc;      // the textual description
        };

        // clean
        class ClassName {
            std::string description;
        };
        ```


## Class Names
Classes or objects should have **noun** or **noun phrase** names
```c++
// bad
Customer getCustomer;

// clean
Customer newCustomer;
```
- their name should **not be a verb**
- moreover, **avoid** **noise words** like `Manager`, `Processor`, `Data` or `Info` in the name of a class
    * if they are **redundant**


## Function Names
Functions should have **verb** or **verb phrase** names
```c++
// bad
monster();

// clean
spawnMonster();
```
- `get`, `set`, `is` are the preferred options


## Problem Domain or Solution Domain
- When you communicate with **programmers**, try to use **solution domain** names
    * such as terms related to computer science, algorithm, pattern, math and so forth
- otherwise, use **problem domain** names


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}