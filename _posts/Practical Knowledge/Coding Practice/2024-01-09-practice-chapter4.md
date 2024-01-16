---
title: "Clean Code : Chapter 4"

categories:
    - coding-practice

tags:
    - [Clean Code, Coding Practice, Comment]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-9
---

# Comments

> 이 포스트는 Clean Code(1st Edition)를 바탕으로 작성되었습니다.

## Self-Explained Code
The prime objective regarding **comments** is to write a code which **explains itself** so that it does **not need any comment**
```c++
// bad
// check to see if the employee is eligible for full benefits
if ((employee.flags & HOURLY_FLAG) && (employee.age > 65))

// clean
if(employee.isEligibleForFullBenefits())
```
- **comments** should explain something that the code can't explain 
- **clean** code with **few** comments is far **superior** to 
    * **bad** code with **lots of** comments
- if you need to write **lots of** comments, then it's a **sign** that
    * you need to **clean your bad** code so that you **need few** comments only


## Good Comments
Although the only truely **good comment** is the comment you found a way **not to write**, there are some cases where **some comments are necessary** or **beneficial** 

### Legal Comments
**Copyright** and **authorship** statements are necessary and reasonable things to put into a comment at the start of each source file
- however, comments like this should **not be contracts** or **legal tomes**
    * where possible, refer to **standard license** or other **external document** rather tha putting all the terms and conditions into the comment

### Informative Comments
If the code is **not clean** enough to explain itself, we may write the **comments** to explain it
```c++
// regular expression
// one or more alphabetic character(s) 
pattern = "[[:alpha:]]+";              

// return value of pure virtual function
class Animal {
public:
    // returns the pointer which points to dynamically allocated object which inherits from Animal class
    virtual Animal* spawnAnimal() = 0;
};
```    
- but still **cleaning** code to **eliminate redundant comments** is the prime objective
- moreover, if the **argument** or **return value** is **obscure** and it's impossbile to alter it
    * then, it's the case where the comment is necessary
    * in this case, it should be **correct**
        + otherwise, there's a chance of the **inappropriate** use

### Explanation of Intent
If you made an **interesting** decision which other programmers might **disagree** with
- it's better to make comments which explains **why you made such decision**

### Warning of Consequences
Sometimes it's useful to **warn** other programmers about certain **consequences**
```c++
// do not run unless you
// have some time to kill
void testWithReallyBigFile() {
    // some codes
}

// or
class DateFormat {
public:
    static SimpleDateFormat makeStandardHttpDateFormat() {
        // SimpleDateFormat is not thread safe,
        // so we need to create each instance independently
        SimpleDateFormat dateFormat = new SimpleDateFormat("EEE, dd MMM   yyy HH:mm:ss z");
        dateFormat.setTimeZone(TimeZone::GMT);
        return dateFormat;
    }
    // other members
};
``` 

### TODO Comments
It is sometimes reasonable to leave **"To do"** notes in the form of `//TODO` comments
- `TODO`s are jobs that the programmer thinks **should be done**, but for some reason **can't do at the moment**
    * it might be a **reminder** to delete a deprecated feature or a **request** for someone else to look at a problem
    * it might be a **request** for someone else to think of a better name or a **reminder** to make a change that is dependent on a planned event
- although there might be some `//TODO` comments, it should **not be an excuse** to leave **bad** code in the system

### Amplication
A comment may be used to **amplify** the **importance** of something that may otherwise seem **inconsequential**


## Bad Comments

### Mumbling
If you **decide to write** a comment, then **spend the time** necessary to make sure it is the **best comment** you can write
- moreover, the **location** of a comment should be **near the code** that the comment explains
- if the comment does **not explain** something **clearly**, it's just a **mess**

### Rudundant Comments
If the code is **self-explained**, then there's **no need** for **comments**
```c++
// bad
// the processor delay
int backgroundProcessorDelay = -1;

// the event listeners
std::vector listeners;
```
- if there are comments for it, it's probably **redundant**

### Mandated Comments 
It is just plain **silly** to have a rule that **every** function or object must have a comment
```c++
// bad
// this function returns the factorial
unsigned long long getFactorial(unsinged n);
```
- if the code is already **self-explained**, then the comments would be **redundant**

### Journal Comments
A **Journal** comment is a comment which **accumulates logs** of every change that has ever been made
- this kind of comments can **be eliminated** by using **Version Control Systems** (such as `git`)


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}