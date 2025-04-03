---
title: "Project 1: Threads"

categories:
    - pintos-1

tags:
    - [Operating Systems, PintOS, Linux, Threads, Scheduling, sleep]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2025-04-03
---

# PintOS : Project 1

## Overview

In this project, I implemented core threading functionality in PintOS, a teaching operating system. The goal was to get a basic scheduler running and handle thread synchronization—pretty foundational stuff, but it taught me a lot about how operating systems manage concurrency.

## Key Features

- **Priority Scheduling**: Built a priority-based scheduler to ensure high-priority threads run first.
- **Synchronization Primitives**: Added locks, semaphores, and condition variables to prevent race conditions.
- **Alarm Clock**: Modified the timer to wake up sleeping threads efficiently.

## Challenges

- Figuring out how to avoid priority inversion was tricky. I ended up implementing priority donation, which was a real "aha!" moment.
- Debugging deadlocks made me question my life choices, but stepping through with GDB saved the day.

## What I Learned

- How thread switching actually works under the hood.
- The importance of testing edge cases (like what happens when 100 threads try to lock the same resource).
- Patience. So much patience.

## Code Snippet

<details>
<summary>Click to see my priority donation code</summary>

```c
void donate_priority(struct thread *donor, struct thread *receiver) {
    if (donor->priority > receiver->priority) {
        receiver->priority = donor->priority;
    }
}
```
</details>

## temp



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}