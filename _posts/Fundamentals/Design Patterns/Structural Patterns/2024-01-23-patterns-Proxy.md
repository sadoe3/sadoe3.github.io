---
title: "Design Patterns : Proxy"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Structural Pattern, Proxy]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-23
---

# Structural Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Proxy

### Also Known As
Surrogate


## Problem

### Intent
Provide a surrogate or placeholder for another object to control access to it.

### Applicability
Proxy is applicable whenever there is a need for a more versatile or sophisticated reference to an object than a simple pointer.
- A **remote proxy** provides a local representative for an object in a different address space.
- A **virtual proxy** creates expensive objects on demand.
- A **protection proxy** controls access to the original object.
    * Protection proxies are use ful when objects should have different access rights.
- A **smart reference** is a replacement for a bare pointer that performs additional actions when an object is accessed.
    * Typical uses include
        + counting the number of references to the real object so that it can be freed automatically when there are no more references (also called [**smart pointers**](https://sadoe3.github.io/cpp/primer-chapter12/#smart-pointers)).
        + loading a persistent object into memory when it's first referenced.
        + checking that the real object is locked before it's accessed to ensure that no other object can change it.

## Solution

### Motivation
Suppose that ~
- The **problem** is that ~
- A **solution** is to ~

### Participants
- **Proxy** (`ImageProxy`)
    * maintains a reference that lets the proxy access the real subject. Proxy may refer to a Subject if the RealSubject and Subject interfaces are the same.
    * provides an interface identical to Subject's so that a proxy can by substituted for the real subject.
    * controls access to the real subject and may be responsible for creating and deleting it.
    * other responsibilities depend on the kind of proxy:
        + *remote proxies* are responsible for encoding a request and its arguments and for sending the encoded request to the real subject in a different address space.
        + *virtual proxies* may cache additional information about the real subject so that they can postpone accessing it.
        + *protection proxies* check that the caller has the access permissions required to perform a request.
- **Subject** (`Graphic`)
    * defines the common interface for RealSubject and Proxy so that a Proxy can be used anywhere a RealSubject is expected.
- **RealSubject** (`Image`)
    * defines the real object that the proxy represents.


### Sample Code
```c++

```

### Implementation
The Proxy pattern can exploit the following language features:
1. *Overloading the member access operator* in C++
    * C++ supports overloading `operator->`, the member access operator.
        + Overloading this operator lets you perform additional work whenever an object is dereferenced.
        + This can be helpful for implementing some kinds of proxy; the proxy behaves just like a pointer.
    * The following example illustrates how to use this technique to implement a virtual proxy called `ImagePtr`.
        ```c++
        class Image;
        extern Image* LoadAnImageFile(const char*);
            // external function

        class ImagePtr {
        public:
            ImagePtr(const char* imageFile);
            virtual ImagePtr();
            
            virtual Image* operator->();
            virtual Image& operator*();
        private:
            Image* LoadImage();
        private:
            Image* _image;
            const char* _imageFile;
        }:

        ImagePtr::ImagePtr(const char* theImageFile) {
            _imageFile = theImageFile;
            _image = 0;
        }

        Image* ImagePtr::LoadImage() {
            if(_image == 0)
                _image = LoadAnImageFile(_imageFile);
            return _ image;
        }
        ```
    * The overloaded `->` and `*` operators use `LoadImage` to return `_image` to callers (loading it if necessary).
        ```c++
        Image* ImagePtr::operator->() {
            return LoadImage();
        } 

        Images& ImagePtr::operator*() {
            return *LoadImage();
        }
        ```
    * This approach lets you call `Image` operations through `ImagePtr` objects without going to the trouble of making the operations part of the `ImagePtr` interface:
        ```c++
        ImagePtr image = ImagePtr("anImageFileName");
        image->Draw(Point(50, 100));
            // (image. operator->())->Draw(Point(50, 100))
        ```
    * Notice how the image proxy acts like a pointer, but it's not declared to be a pointer to an Image. That means you can't use it exactly like a real pointer to an Image. Hence clients must treat Image and ImagePtr objects differently in this approach.
    * Overloading the member access operator isn't a good solution for every kind of proxy. Some proxies need to know precisely which operation is called, and overloading the member access operator doesn't work in those cases.
    * Consider the virtual proxy example in the Motivation. The image should be loaded at a specific time namely when the Draw operation is called-and not whenever the image is referenced. Overloading the access operator doesn't allow this distinction. In that case we must manually implement each proxy operation that forwards the request to the subject.
    * These operations are usually very similar to each other, as the Sample Code demonstrates. Typically all operations verify that the request is legal, that the original object exists, etc., before forwarding the request to the subject. It's tedious to write this code again and again. So it's common to use a preprocessor to generate it automatically.
2. *Proxy doesn't always have to know the type of real subject*
    * If a Proxy class can deal with its subject solely through an abstract interface, then there's no need to make a Proxy class for each RealSubject class; the proxy can deal with all RealSubject classes uniformly. But if Proxies are going to instantiate RealSubjects (such as in a virtual proxy), then they have to know the concrete class.

### Related Patterns
- **Adapter**: An adapter provides a different interface to the object it adapts.
    * In contrast, a proxy provides the same interface as its subject.
    * However, a proxy used for access protection might refuse to perform an operation that the subject will perform
        + so its interface may be effectively a subset of the subject's.
- **Decorator**: Although decorators can have similar implementations as proxies, decorators have a different purpose.
    * A decorator adds one or more responsibilities to an object, whereas a proxy controis access to an object.
- Proxies vary in the degree to which they are implemented like a decorator.
    * A protection proxy might be implemented exactly like a decorator. On the other hand, a remote proxy will not contain a direct reference to its real subject but only an indirect refer-ence, such as "host ID and local address on host." A virtual proxy will start off with an indirect reference such as a file name but will eventually obtain and use a direct reference.


## Consequences
The Proxy pattern introduces a level of indirection when accessing an object. The additional indirection has many uses, depending on the kind of proxy:
1. A remote proxy can hide the fact that an object resides in a different address space.
2. A virtual proxy can perform optimizations such as creating an object on demand.
3. Both protection proxies and smart references allow additional housekeeping tasks when an object is accessed.
- There's another optimization that the Proxy pattern can hide from the client. It's called **copy-on-write**, and it's related to creation on demand. Copying a large and complicated object can be an expensive operation. If the copy is never modified, then there's no need to incur this cost. By using a proxy to postpone the copying process, we ensure that we pay the price of copying the object only if it's modified.
- To make copy-on-write work, the subject must be reference counted. Copying the proxy will do nothing more than increment this reference count. Only when the client requests an operation that modifies the subject does the proxy actually copy it. In that case the proxy must also decrement the subject's reference count. When the reference count goes to zero, the subject gets deleted.
- Copy-on-write can reduce the cost of copying heavyweight subjects significantly.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}