---
title: "Design Patterns : Chain of Responsibility"

categories:
    - design-patterns

tags:
    - [Design Patterns, OOP, Behavioral Pattern, Chain of Responsibility]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-1-23
---

# Behavioral Pattern

> 이 포스트는 Design Patterns (1st Edition)를 바탕으로 작성되었습니다.

## Pattern Name
Chain of Responsibility


## Problem

### Intent
Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it.

### Applicability
Use **Chain of Responsibility** when
- more than one object may handle a request, and the handler isn't known a *priori*.
    * The handler should be ascertained automatically.
- you want to issue a request to one of several objects without specifying the receiver explicitly.
- the set of objects that can handle a request should be specified dynamically.


## Solution

### Motivation
Suppose that ~
- The **problem** is that ~
- A **solution** is to ~

### Participants
- **Handler** (`HelpHandler`)
    * defines an interface for handling requests.
    * (optional) implements the successor link.
- **ConcreteHandler** (`PrintButton`, `PrintDialog`)
    * handles requests it is responsible for.
    * can access its successor.
    * if the ConcreteHandler can handle the request, it does so; otherwise it forwards the request to its successor.
- **Client**
    * initiates the request to a ConcreteHandler object on the chain.

### Sample Code
```c++

```

### Implementation
Here are implementation issues to consider in Chain of Responsibility:
1. *Implementing the successor chain*
    * There are two possible ways to implement the successor chain:
        + (a) Define new links (usually in the Handler, but ConcreteHandlers could define them instead).
        + (b) Use existing links.
    * Our examples so far define new links, but often you can use existing object references to form the successor chain. For example, parent references in a part-whole hierarchy can define a part's successor. A widget structure might already have such links. Composite discusses parent references in more detail.
    * Using existing links works well when the links support the chain you need. It saves you from defining links explicitly, and it saves space. But if the structure doesn't reflect the chain of responsibility your application requires, then you'll have to define redundant links.
2. *Connecting successors*
    * If there are no preexisting references for defining a chain, then you'll have to introduce them yourself. In that case, the Handler not only defines the interface for the requests but usually maintains the successor as well. That lets the handler provide a default implementation of HandleRequest that forwards the request to the successor (if any). If a ConcreteHandler subclass isn't interested in the request, it doesn't have to override the forwarding operation, since its default implementation forwards unconditionally.
    * Here's a HelpHandler base class that maintains a successor link:
        ```c++
        class HelpHandler { 
        public:
            HelpHandler(HelpHandler* s) :_successor (s) { }
            virtual void HandleHelp();
        private:
            HelpHandler* _successor;
        };

        void HelpHandler::HandleHelp() {
            if(successor) 
                successor->HandleHelp();
        }
        ```
3. *Representing requests*
    * Different options are available for representing requests. In the simplest form, the request is a hard-coded operation invocation, as in the case of HandleHelp. This is convenient and safe, but you can forward only the fixed set of requests that the Handler class defines.
An alternative is to use a single handler function that takes a request code(e.g., an integer constant or a string) as parameter. This supports an open-ended set of requests. The only requirement is that the sender and receiver agree on how the request should be encoded.
This approach is more flexible, but it requires conditional statements for dispatching the request based on its code. Moreover, there's no type-safe way to pass parameters, so they must be packed and unpacked manually. Obviously this is less safe than invoking an operation directly.
Lo address the parameter-passing problem, we can use separate request objects that bundle request parameters. A Request class can represent requests explic-itly, and new kinds of requests can be defined by subclassing. Subclasses can define different parameters. Handlers must know the kind of request (that is, Which Request subclass they're using) to access these parameters.
To identify the request, Request can define an accessor function that returns an identifier for the class. Alternatively, the receiver can use run-time type information if the implementation languages supports it.
    * Here is a sketch of a dispatch function that uses request objects to identify requests. A GetKind operation defined in the base Request class identifies the kind of request:
        ```c++
        void Handler::HandleRequest(Request* theRequest) {
            switch(theRequest->GetKind()) {
            case Help:
                // cast argument to appropriate type
                HandleHelp((HelpRequest*) theRequest);
                break;
            case Print:
                HandlePrint((PrintRequest*) theRequest);
                //...
                break;
            default:
                //...
                break;
            } 
        }
        ```
    * Subclasses can extend the dispatch by overriding HandleRequest. The subclass handles only the requests in which it's interested; other requests are forwarded to the parent class. In this way, subclasses effectively extend (rather than override) the HandleRequest operation. For example, here's how an ExtendedHandler subclass extends Handler's version of HandleRequest:
        ```c++
        class ExtendedHandler : public Handler {
        public:
            virtual void HandleRequest(Request* theRequest);
            //...
        };

        void ExtendedHandler::HandleRequest(Request* theRequest) {
            switch(theRequest->GetKind()) {
            case Preview:
                // handle the Preview request
                break;
            default:
                // let Handler handle other requests
                Handler:: HandleRequest(theRequest);
            } 
        }
        ```

### Related Patterns
- Chain of Responsibility is often applied in conjunction with **Composite**.
    * There, a component's parent can act as its successor.


## Consequences
Chain of Responsibility has the following benefits and liabilities:
1. *Reduced coupling*
    * The pattern frees an object from knowing which other object handles a request. An object only has to know that a request will be handled "appro-priately." Both the receiver and the sender have no explicit knowledge of each other, and an object in the chain doesn't have to know about the chain's structure.
As a result, Chain of Responsibility can simplify object interconnections. Instead of objects maintaining references to all candidate receivers, they keep a single reference to their successor.
2. *Added flexibility in assigning responsibilities to objects*
    * Chain of Responsibility gives you added flexibility in distributing responsibilities among objects. You can add or change responsibilities for handling a request by adding to or otherwise changing the chain at run-time. You can combine this with subclassing to specialize handlers statically.
3. *Receipt isn't guaranteed*
    * Since a request has no explicit receiver, there's no guarantee it'll be handled-the request can fall off the end of the chain without ever being handled. A request can also go unhandled when the chain is not configured properly.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}