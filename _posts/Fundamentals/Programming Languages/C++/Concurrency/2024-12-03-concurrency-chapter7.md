---
title: "C++ Concurrency : Lock-Free Data Structures"

categories:
    - cpp

tags:
    - [C++, Programming Language, Concurrency, lock-free, atomic, data structure]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-12-03
---

# C++ Concurrency : Chapter 7

> 이 포스트는 `C++ Concurrency in Action (2nd ed.)`을 바탕으로 작성되었습니다.

## Definition of Lock-Free
Algorithms and data structures that use `mutexes`, `condition variables`, and `futures` to **synchronize** access to data are classified as **`blocking`** data structures and algorithms
- This is because a `thread` is completely **blocked** until it is unblocked by the appropriate action of another thread
- Data structures and algorithms that do **not** rely on **blocking** library functions are called **`non-blocking`**
  - However, it is important to note that **not** all **`non-blocking`** data structures are **`lock-free`**

### Types of Non-Blocking Data Structures
There are **3 types** of `non-blocking` data structures
- **Obstruction-Free**
    * If all other threads are paused, any given thread will complete its operation in a bounded number of steps
    * This type is generally not very useful
        + Because it is rare for all other threads to be paused
    * Thus, it is typically a characterization of a **failed** `lock-free` implementation.
- **Lock-Free**
    * If multiple threads are operating on a data structure, then after a bounded number of steps, one of them will complete its operation
    * For a data structure to qualify as `lock-free`, more than one thread must be **able** to access the structure **concurrently**
- **Wait-Free**
    * A `wait-free` data structure is a `lock-free` data structure with the **additional property** that
        + Every thread operating on the data structure will complete its operation in a bounded number of steps, even if other threads are also operating on it
    * To achieve this, the following must be ensured
        + Each operation can be performed in a single pass
        + The steps performed by one thread should not cause the operation of another thread to fail

### Pros and Cons of Lock-Free Data Structures
Given the **difficulty** of correctly implementing `lock-free` or `wait-free` data structures, it is important to ensure that the benefits outweigh the costs
- **Pros**
    * **Maximized Concurrency**
        + `Lock-free` data structures allow some threads to make progress with every step
        + `Wait-free` data structures enable every thread to make progress because no waiting is necessary
    * **Robustness**
        + If a thread dies while holding a lock in a traditional data structure, the structure may become broken permanently
        + In a l`ock-free` data structure, if a thread dies midway through an operation, only that thread's data is lost, and other threads can proceed normally
    * **No Deadlocks**
        + Since **no** `locks` are used, `deadlocks` are **impossible**
        + However, **`live-locks`** can occur instead
- **Cons**
    * **Live-Lock**
        + A `live-lock` occurs when two threads each attempt to modify the data structure, but the changes made by each thread require the other's operation to be restarted
        + As a result, both threads loop and repeatedly try again without making progress
    * **Complexity**
        + While the features provided by `wait-free` data structures are desirable, they are difficult to achieve
        + Implementing them requires more complex code and may involve additional steps for the same action
    * **Performance**
        + Although `lock-free` data structures enable maximum concurrency, they **may decrease** overall performance
        + This is because atomic operations can be much slower than non-atomic operations, and there are likely to be more atomic operations in a lock-free data structure
        + The hardware must synchronize data between threads that access the same atomic variable, which can incur significant performance overhead

### Performance Considerations
As with any approach, it is important to **evaluate** the relevant **performance** aspects **before** committing to `lock-based` or `lock-free` data structures
- Key factors to consider include
    * `Worst-case` waiting time
    * `Average` wait time
    * `Overall` execution time
    * Or any other relevant performance metrics


## A Thread-Safe Stack without Locks
Now let's implement a `stack` **without locks**

### Simple Implementation
In order to fully achieve `lock-free` functionality, it is essential to understand the underlying mechanics of how `lock-free` data structures use `atomic operations` for **synchronization**
```cpp
template<typename T>
class StackLockFree {
    struct Node {
		T data;
		Node* next;
		Node(const T& inputData) : data(inputData) {}
	};
public:
	void push(const T& data) {
		Node* const newNode = new Node(data);
		newNode->next = head.load();
		while (!head.compare_exchange_weak(newNode->next, newNode))
			;
	}
private:
	std::atomic<Node*> head;
};
```
- The core concept here is to use the `.compare_exchange_weak()` method on a `std::atomic<Node*>` object for synchronization

### Exception Handling
Due to **exception-safety** concerns, it's preferable to use `std::shared_ptr<T>` instead of directly using `T` for `data`
```cpp
template<typename T>
class StackLockFree {
    struct Node {
        std::shared_ptr<T> data;
        Node* next;
        Node(const T& inputData) : data(std::make_shared<T>(inputData)) { }
    };
public:
    void push(const T& data) {
        Node* const newNode = new Node(data);
        newNode->next = head.load();
        while (!head.compare_exchange_weak(newNode->next, newNode))
            ;
    }
    std::shared_ptr<T> pop() {
        Node* oldHead = head.load();
        while (oldHead && !head.compare_exchange_weak(oldHead, oldHead->next))
            ;
        return oldHead ? oldHead->data : std::shared_ptr<T>();
    }
private:
    std::atomic<Node*> head;
};
```
- This approach is **exception-safe**
    * However, as you've noticed, the premature stack poses a **`memory leak`**
    * This is because `pop()` only returns the `data` and does **not deallocate** the `oldHead`

### Without Memory Leaks
The solution is simple: place the node to be deleted in a list and then delete the nodes in the list when only a single thread calls `pop()`
```cpp
template<typename T>
class StackLockFree {
    struct Node
    {
        std::shared_ptr<T> data;
        Node* next;
        Node(const T& inputData) : data(std::make_shared<T>(inputData)) {}
    };
public:
    void push(const T& data) {
        Node* const newNode = new Node(data);
        newNode->next = head.load();
        while (!head.compare_exchange_weak(newNode->next, newNode))
            ;
    }
    std::shared_ptr<T> pop() {
        ++numberThreadsInPop;
        
        Node* oldHead = head.load();
        while (oldHead && !head.compare_exchange_weak(oldHead, oldHead->next))
            ;
        std::shared_ptr<T> extractedData;
        if (oldHead)
            extractedData.swap(oldHead->data);    // extract the data from the Node to be deleted

        tryDeallocate(oldHead);
        return extractedData;                     // return the extracted data
    }
private:
    std::atomic<Node*> head;
    std::atomic<unsigned> numberThreadsInPop;
    std::atomic<Node*> nodeToBeDeleted;
    static void deleteNodes(Node* nodes) {
        while (nodes) {
            Node* next = nodes->next;
            delete nodes;
            nodes = next;
        }
    }
    void tryDeallocate(Node* oldHead) {
        if (numberThreadsInPop == 1) {
            Node* targetNode = nodeToBeDeleted.exchange(nullptr);
            if (--numberThreadsInPop == 0)
                deleteNodes(targetNode);
            else if (targetNode)
                chainPendingNodes(targetNode);
            delete oldHead;
        }
        else {
            chainPendingNode(oldHead);
            --numberThreadsInPop;
        }
    }
    void chainPendingNodes(Node* nodes) {
        Node* last = nodes;
        Node* next = nullptr;
        while (next = last->next)
            last = next;
        chainPendingNodes(nodes, last);
    }
    void chainPendingNodes(Node* first, Node* last) {
        last->next = nodeToBeDeleted;
        while (!nodeToBeDeleted.compare_exchange_weak(last->next, first));
    }
    void chainPendingNode(Node* n) {
        chainPendingNodes(n, n);
    }
};
```
- As you see, an `atomic counter` is utilized to track the number of threads interacting with the node
- It's worth noting that this implementation works well for **low-load** situations
    * In **high-load** situations, there may **never** be a state where `numberThreadsInPop` equals `1`
    * This results in an inability to deallocate, causing a **`memory leak`**
- Therefore, you need to upgrade this approach, and there are two available options
    * **Hazard Pointers**
    * **Reference Counting**


## Hazard Pointer
The `hazard pointer` acts as a `mark` that identifies which object a thread is actively working on, helping to ensure that the object **isn't deallocated** while the thread is **still using it**
- They are typically stored in a location that is **accessible to all threads**
- When a thread **accesses** a node in a concurrent data structure, it **sets** a `hazard pointer` to that node to signal that it's **actively using it**
   * This **prevents** other threads from **deallocating** the node during the time it is being used by the thread
   * In other words, the `hazard pointer` serves as a `protection mechanism`
- Once the thread **finishes** its work with the node (i.e., when the thread is done accessing or modifying the node), it **clears** its `hazard pointer`
   - This signals that the node is **no longer in use** by the thread, and it **can** be safely **deallocated** by other threads if needed
- If another thread tries to **deallocate** the node (for example, as part of a `pop()` operation or some other cleanup)
    * it **first checks** if there are any `hazard pointers` pointing to the node
    * If a `hazard pointer` is **set** (indicating that another thread is **using** the node)
        + the node is **not deallocated immediately**
        + Instead, the node is **added** to a `deallocation list`, meaning it will be considered for **deallocation later**, once it's safe
    * Otherwise (meaning that there's **no** thread **using** the node)
        + the node is **deallocated immediately**
- Additionally, whenever `pop()` operation is called, the thread also **checks** if there are any nodes which have **no `hazard pointer`**  from the `deallocation list` 
    * If there are, the thread **deallocates** those nodes safely

### Upgarded `pop()`
The code below shows the upgraded version of `pop()` with a `hazard pointer`
```cpp
template<typename T>
std::shared_ptr<T> StackLockFree<T>::pop() {
    std::atomic<void*>& hazardPointer = getHazardPointerThisThread();
    Node* oldHead = head.load();
    do {
        Node* temp;
        do {
            temp = oldHead;
            hazardPointer.store(oldHead);   // set the hazard pointer to oldHead so that other threads don't delete it 
            oldHead = head.load();
        } while (oldHead != temp);
    } while (oldHead && !head.compare_exchange_strong(oldHead, oldHead->next));
    hazardPointer.store(nullptr);

    std::shared_ptr<T> extractedData;
    if (oldHead) {
        extractedData.swap(oldHead->data);
        if (isOtherHazardPointerRemaining(oldHead))
            deallocateLater(oldHead);
        else
            delete oldHead;
        deallocateNodesWithNoHazardPointers();
    }
    return extractedData;
}
```
- The **details** of `hazard pointer` operations will be covered **below**
    * For now, it is sufficient to understand how it works
- It's worth noting that `compare_exchange_strong()` is used to **avoid** *`spurious failures`*

### `getHazardPointerThisThread()`
The code below shows the how `getHazardPointerThisThread()` is implemented
```cpp
constexpr unsigned MAX_HAZARD_POINTERS = 100;       // arbitrary number
struct HazardPointer {
    std::atomic<std::thread::id> id;
    std::atomic<void*> pointer;
};
HazardPointer hazardPointers[MAX_HAZARD_POINTERS];
class hazardPointerOwner {
public:
    hazardPointerOwner(const hazardPointerOwner&) = delete;
    hazardPointerOwner operator=(const hazardPointerOwner&) = delete;
    hazardPointerOwner() : hazardPointer(nullptr) {
        for (unsigned i = 0; i < MAX_HAZARD_POINTERS; ++i) {
            std::thread::id oldID;
            if (hazardPointers[i].id.compare_exchange_strong(oldID, std::this_thread::get_id())) {
                hazardPointer = &hazardPointers[i];
                break;
            }
        }
        if (!hazardPointer)
            throw std::runtime_error("No hazard pointers available");
    }
    std::atomic<void*>& getPointer() {
        return hazardPointer->pointer;
    }
    ~hazardPointerOwner() {
        hazardPointer->pointer.store(nullptr);
        hazardPointer->id.store(std::thread::id());
    }
private:
    HazardPointer* hazardPointer;
};
std::atomic<void*>& getHazardPointerThisThread() {
    static thread_local hazardPointerOwner hazard;
    return hazard.get_pointer();
}
```
- It's worth noting that you can **improve** the solution by changing the `location` or `length of the array`
    * This example only demonstrates the raw implementation as a tutorial
- It's also crucial to understand that a `static thread_local` object **exists separately** for **each thread**
    * It is **created** when the `thread` **starts**
    * It is **destroyed** when the `thread` **terminates**

### Other Operations for Hazard Pointers
```cpp
bool isOtherHazardPointerRemaining(void* p) {
    for (unsigned i = 0; i < MAX_HAZARD_POINTERS; ++i) {
        if (hazardPointers[i].pointer.load() == p)
            return true;
    }
    return false;
}
```
```cpp
// deallocate functions
template<typename T>
void doDeallocate(void* p) {
    delete static_cast<T*>(p);
}
struct DataToDeallocate {
    template<typename T>
    DataToDeallocate(T* p) : data(p), deleter(&doDeallocate<T>), next(0) {}
    ~DataToDeallocate() {
        deleter(data);
    }

    void* data;
    std::function<void(void*)> deleter;
    DataToDeallocate* next;
};
std::atomic<DataToDeallocate*> nodesToDeallocate;
void addToDeallocateList(DataToDeallocate* node) {
    node->next = nodesToDeallocate.load();
    while (!nodesToDeallocate.compare_exchange_weak(node->next, node))
        ;
}
template<typename T>
void deallocateLater(T* data) {
    addToDeallocateList(new DataToDeallocate(data));
}
void deallocateNodesWithNoHazardPointers() {
    DataToDeallocate* current = nodesToDeallocate.exchange(nullptr);
    while (current) {
        DataToDeallocate* const next = current->next;
        if (!isOtherHazardPointerRemaining(current->data))
            delete current;
        else
            addToDeallocateList(current);
        current = next;
    }
}
```

### Example Use
```cpp
template<typename T>
void readStack(StackLockFree<T>& stack) {
    std::shared_ptr<T> emptyData;
    while (true) {
        std::shared_ptr<T> newData = stack.pop();
        if(newData != emptyData)
            std::cout << *newData << " is read" << std::endl;
    }
}
template<typename T>
void writeStack(StackLockFree<T>& stack, const T& inputValue) {
    stack.push(inputValue);
}

int main() {
    StackLockFree<int> stack;
    std::thread t1(readStack<int>, std::ref(stack));
    std::thread t2(writeStack<int>, std::ref(stack), 3);
    std::thread t3(writeStack<int>, std::ref(stack), -1);
    
    t1.join();
    t2.join();
    t3.join();

    return 0;
}
/*
possible outcome
-1 is read
3 is read
*/
```


## Reference Counting
While `harzard pointer` stores a list of the nodes in use, `reference counting` stores a count of the number of threads accessing each node

### Example
```cpp
template<typename T>
class StackLockFree {
    struct Node;
    struct CountedNodePointer {
        int externalCount;
        Node* pointer;
    };
    struct Node {
        std::shared_ptr<T> data;
        std::atomic<int> internalCount;
        CountedNodePointer next;
        Node(const T& inputData) : data(std::make_shared<T>(inputData)), internalCount(0) { }
    };
public:
    ~StackLockFree() {
        while (pop())
            ;
    }
    void push(const T& data) {
        CountedNodePointer newNode;
        newNode.pointer = new Node(data);
        newNode.externalCount = 1;
        newNode.pointer->next = head.load();
        while (!head.compare_exchange_weak(newNode.pointer->next, newNode))
            ;
    }
    std::shared_ptr<T> pop() {
        CountedNodePointer oldHead = head.load();
        while (true) {
            increaseHeadCount(oldHead);              // increase the counter so that it's safe to dereference oldHead
            Node* const pointer = oldHead.pointer;
            if (!pointer)
                return std::shared_ptr<T>();

            if (head.compare_exchange_strong(oldHead, pointer->next)) {
                std::shared_ptr<T> extractedData;
                extractedData.swap(pointer->data);
                const int COUNT_INCREASE = oldHead.externalCount - 2;
                if (pointer->internalCount.fetch_add(COUNT_INCREASE) == -COUNT_INCREASE)
                    delete pointer;
                return extractedData;
            }
            else if (pointer->internalCount.fetch_sub(1) == 1)
                delete pointer;
        }
    }
private:
    void increaseHeadCount(CountedNodePointer& oldCounter) {
        CountedNodePointer newCounter;
        do {
            newCounter = oldCounter;
            ++newCounter.externalCount;
        } while (!head.compare_exchange_strong(oldCounter, newCounter));
        oldCounter.externalCount = newCounter.externalCount;
    }

    std::atomic<CountedNodePointer> head;
};
/*
possible outcome with same use case
-1 is read
3 is read
*/
```
- `push()` sets up the
    * `externalCount` as `1`
        + because the `head` pointer points to, then the newly added node will point to it
    * and the `internalCount` as `0`
- `pop()` **increases** the `externalCount`
    * so that it's **safe** to **dereference** it
- Then, `pop()` checks whether `head` is synchronized with `oldHead`
    * If it is, then it calls `.fetch_add()` with `oldHead.externalCount - 2` on `internalCount`
        + `-2` is applied because **one** is for removing the `existing node`, and **another one** is for decreasing the `count` from `this thread`
        + if the `previous value` of `internalCount` was **equal** to the `negative value` of `oldHead.externalCount - 2`
            - then, this means that **only** this thread is accessing the node, hence it's **safe** to **delete** it
        + otherwise, delete it **later**
    * Otherwise, it calls `.fetch_sub()` with `1` on `internalCount`
        + because it needs to decrease the `count` from `this thread`
        + and if the `previous value` of `internalCount` was **equal** to `1`
            - then, this means that **only** this thread is accessing the node, hence it's **safe** to **delete** it
        + otherwise, delete it **later**

### Applying Other Memory Models
```cpp
template<typename T>
class StackLockFree {
    struct Node;
    struct CountedNodePointer {
        int externalCount;
        Node* pointer;
    };
    struct Node {
        std::shared_ptr<T> data;
        std::atomic<int> internalCount;
        CountedNodePointer next;
        Node(const T& inputData) : data(std::make_shared<T>(inputData)), internalCount(0) {}
    };

public:
    ~StackLockFree() {
        while (pop())
            ;
    }
    void push(const T& data) {
        CountedNodePointer newNode;
        newNode.pointer = new Node(data);
        newNode.externalCount = 1;
        newNode.pointer->next = head.load(std::memory_order_relaxed);
        while (!head.compare_exchange_weak(newNode.pointer->next, newNode, std::memory_order_release, std::memory_order_relaxed))
            ;
    }
    std::shared_ptr<T> pop() {
        CountedNodePointer oldHead = head.load(std::memory_order_relaxed);
        while(true) {
            increaseHeadCount(oldHead);
            Node* const pointer = oldHead.pointer;
            if (!pointer) 
                return std::shared_ptr<T>();

            if (head.compare_exchange_strong(oldHead, pointer->next, std::memory_order_relaxed)) {
                std::shared_ptr<T> extractedData;
                extractedData.swap(pointer->data);
                const int COUNT_INCREASE = oldHead.externalCount - 2;
                if (pointer->internalCount.fetch_add(COUNT_INCREASE, std::memory_order_release) == -COUNT_INCREASE)     // ensures swap() happens before delete
                    delete pointer;
                return extractedData;
            }
            else if (pointer->internalCount.fetch_add(-1, std::memory_order_relaxed) == 1) {
                pointer->internalCount.load(std::memory_order_acquire);                                                 // ensures delete happens after fetch_add()
                delete pointer;
            }
        }
    }
private:
    void increaseHeadCount(CountedNodePointer& oldCounter) {
        CountedNodePointer newCounter;
        do {
            newCounter = oldCounter;
            ++newCounter.externalCount;
        } while (!head.compare_exchange_strong(oldCounter, newCounter, std::memory_order_acquire, std::memory_order_relaxed));
        oldCounter.externalCount = newCounter.externalCount;
    }

    std::atomic<CountedNodePointer> head;
};
```
- In order to use `acquire-release` ordering, you need to choose **two operations** for a `synchronization pair`
    * One for `store` (`memory_order_release`)
      + `push()` is appropriate in this case
    * One for `load` (`memory_order_acquire`)  
      + `increaseHeadCount()` is appropriate in this case
- With this `pair`, you can ensure that `push()` **always happens before** `pop()` 
    * at least before `increaseHeadCount()` in `pop()`
- The only remaining operation to deal with is to `delete` the target node
    * You must ensure that the `swap()` **always happens before** `delete` to avoid a data race
      + This can be easily achieved by using `std::memory_order_release` ordering for `fetch_add()`
    * Additionally, you need to ensure that `delete` of the target pointer **happens after** `fetch_add()` in the second case
      + This can be accomplished by calling `load()` with `std::memory_order_acquire` on the same `internalCount` object


## A Thread-Safe Queue without Locks
To implement a `queue`, the fundamental approach is similar
- It involves utilizing `reference counting`
- However, in this case, an **additional pointer**, specifically `tail`, is required

### Example
```cpp
template<typename T>
class QueueLockFree {
    struct Node;
    struct CountedNodePointer {
        int externalCount;
        Node* pointer;
    };
    struct NodeCounter {
        // keep the total counter size to 32 bits so that it works on 32-bit and 64bit machines
        unsigned internalCount : 30;
        unsigned externalCounters : 2;
    };

    struct Node {
        void decreaseInternalCount() {
            NodeCounter oldCounter = count.load(std::memory_order_relaxed);
            NodeCounter newCounter;
            do {
                newCounter = oldCounter;
                --newCounter.internalCount;
            } while (!count.compare_exchange_strong(oldCounter, newCounter, std::memory_order_acquire, std::memory_order_relaxed));
            if (!newCounter.internalCount && !newCounter.externalCounters)
                delete this;
        }

        std::atomic<T*> data;
        std::atomic<NodeCounter> count;
        std::atomic<CountedNodePointer> next;
    };
public:
    QueueLockFree() : head({0, new Node()}), tail(head.load()) {}
    QueueLockFree(const QueueLockFree& other) = delete;
    QueueLockFree& operator=(const QueueLockFree& other) = delete;
    ~QueueLockFree() {
        CountedNodePointer oldHead = head.load();
        while (oldHead.pointer) {
            head.store(oldHead.pointer->next);
            delete oldHead.pointer;
            oldHead = head.load();
        }
    }


    void push(T newValue) {
        std::unique_ptr<T> newData(new T(newValue));
        CountedNodePointer newNext;
        newNext.pointer = new Node;
        newNext.externalCount = 1;
        CountedNodePointer oldTail = tail.load();
        while (true) {
            increaseExternalCount(tail, oldTail);
            T* oldData = nullptr;
            if (oldTail.pointer->data.compare_exchange_strong(oldData, newData.get())) {
                CountedNodePointer oldNext = { 0 };
                if (!oldTail.pointer->next.compare_exchange_strong(oldNext, newNext)) {
                    delete newNext.pointer;
                    newNext = oldNext;
                }
                setNewTail(oldTail, newNext);
                newData.release();
                break;
            }
            else {
                CountedNodePointer oldNext = { 0 };
                if (oldTail.pointer->next.compare_exchange_strong(oldNext, newNext)) {
                    oldNext = newNext;
                    newNext.pointer = new Node;
                }
                setNewTail(oldTail, oldNext);
            }
        }
    }

    std::unique_ptr<T> pop() {
        CountedNodePointer oldHead = head.load(std::memory_order_relaxed);
        while (true) {
            increaseExternalCount(head, oldHead);
            Node* const pointer = oldHead.pointer;
            if (pointer == tail.load().pointer)
                return std::unique_ptr<T>();

            CountedNodePointer next = pointer->next.load();
            if (head.compare_exchange_strong(oldHead, next)) {
                T* const extractedData = pointer->data.exchange(nullptr);
                decreaseExternalCount(oldHead);
                return std::unique_ptr<T>(extractedData);
            }
            pointer->decreaseInternalCount();
        }
    }
private:
    static void increaseExternalCount(std::atomic<CountedNodePointer>& counter, CountedNodePointer& oldCounter) {
        CountedNodePointer newCounter;
        do {
            newCounter = oldCounter;
            ++newCounter.externalCount;
        } while (!counter.compare_exchange_strong(oldCounter, newCounter, std::memory_order_acquire, std::memory_order_relaxed));
        oldCounter.externalCount = newCounter.externalCount;
    }

    static void decreaseExternalCount(CountedNodePointer& oldNodePointer) {
        Node* const pointer = oldNodePointer.pointer;
        const int COUNT_INCREASE = oldNodePointer.externalCount - 2;
        NodeCounter oldCounter = pointer->count.load(std::memory_order_relaxed);
        NodeCounter newCounter;
        do {
            newCounter = oldCounter;
            --newCounter.externalCounters;
            newCounter.internalCount += COUNT_INCREASE;
        } while (!pointer->count.compare_exchange_strong( oldCounter, newCounter, std::memory_order_acquire, std::memory_order_relaxed));
        if (!newCounter.internalCount && !newCounter.externalCounters)
            delete pointer;
    }

    void setNewTail(CountedNodePointer& oldTail, const CountedNodePointer& newTail) {
        Node* const currentTailPointer = oldTail.pointer;
        while (!tail.compare_exchange_weak(oldTail, newTail) && oldTail.pointer == currentTailPointer)
            ;

        if (oldTail.pointer == currentTailPointer)
            decreaseExternalCount(oldTail);
        else
            currentTailPointer->decreaseInternalCount();
    }

    std::atomic<CountedNodePointer> head;
    std::atomic<CountedNodePointer> tail;
};
```
- It's worth noting that `push()` provides a **helping** mechanism
    * It **sets** the `CountedNodePointer` object to `next` on `tail`
    * It starts to **help** when another thread is **already adding** the new node
- `push()` can check this with the following condition
    * `oldTail.pointer->data.compare_exchange_strong(oldData, newData.get())`
        + If it returns `true`, this means it's safe to add
        + Otherwise, you need to **wait** until the other thread has finished its work
            - Hence, you need to **help** it
- Moreover, when adding the node, you need to check whether another thread **has helped** you
    * You can check this with the condition:
        + `!oldTail.pointer->next.compare_exchange_strong(oldNext, newNext)`
        + If this returns `true`, it means the other thread **helped** you
        + Otherwise, you can use your own `CountedNodePointer` object for `next` on `tail` because there's **no help**
- By **helping** another thread, you can **avoid** creating `wait loops` and wasting `CPU time`
- The code below shows the use case of this
    ```cpp
    // use case
    template<typename T>
    void readQueue(QueueLockFree<T>& queue) {
        std::shared_ptr<T> emptyData;
        while (true) {
            std::shared_ptr<T> newData = queue.pop();
            if(newData != emptyData)
                std::cout << *newData << " is read" << std::endl;
        }
    }
    template<typename T>
    void writeQueue(QueueLockFree<T>& queue, const T& inputValue) {
        queue.push(inputValue);
    }

    int main() {
        QueueLockFree<int> queue;
        std::thread t1(readQueue<int>, std::ref(queue));
        std::thread t2(writeQueue<int>, std::ref(queue), 3);
        std::thread t3(writeQueue<int>, std::ref(queue), -1);

        t1.join();
        t2.join();
        t3.join();

        return 0;
    }
    /*
    possible outcome
    -1 is read
    3 is read
    */
    ```


## Guidelines for Writing Lock-Free Data Structures

### Use `std::memory_order_seq_cst` for Prototyping
Using other `memory orderings` is an **`optimization`**
- Hence, it's recommended to **avoid** doing it **prematurely**

### Use a Lock-Free Deallocation Method
- In this chapter, you've explored **three techniques** for ensuring that `memory` can be **safely deallocated**
    * `Waiting` until no threads are accessing the data structure and then deleting all objects that are pending deletion
    * Using `hazard pointers` to track whether a thread is currently accessing a particular object
    * Implementing `reference counting` to ensure that objects are not removed until no references to them remain
- In all cases, the **key idea** is to use a method to **monitor** how many threads are **accessing** a particular object and to **delete** the object **only** when it is **no longer referenced** by any thread

### Watch Out for the ABA problem
The `ABA problem` occurs in concurrent programming when a value is read (`A`), changed (to `B`), and then changed back to its original value (`A`), leading to **incorrect assumptions** that the value **hasn't changed**
- To resolve this, techniques like `ABA counters` or `hazard pointers`, are used to detect and avoid false assumptions about data integrity


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}