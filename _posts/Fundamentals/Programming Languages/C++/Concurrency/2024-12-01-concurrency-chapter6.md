---
title: "C++ Concurrency : Lock-based Data Structures"

categories:
    - cpp

tags:
    - [C++, Programming Language, Concurrency, lock-based, mutex, data structure, stack, queue, list]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-12-01
---

# C++ Concurrency : Chapter 6

> 이 포스트는 `C++ Concurrency in Action (2nd ed.)`을 바탕으로 작성되었습니다.

## Design with a Single Mutex
The design of `lock-based` **concurrent** data structures focuses on ensuring that the appropriate `mutex` is **locked** when **accessing** the data, and that the lock is held for the **minimum** amount of time

### A Thread-Safe Stack
```cpp
#include <stack>
template<typename T>
class stackThreadSafe {
public:
    stackThreadSafe() {}
    stackThreadSafe(const stackThreadSafe& other) {
        std::lock_guard<std::mutex> lock(other.m);
        data = other.data;
    }
    stackThreadSafe& operator=(const stackThreadSafe&) = delete;

    void push(T newValue) {
        std::lock_guard<std::mutex> lock(m);
        data.push(std::move(newValue));
    }
    std::shared_ptr<T> pop() {
        std::lock_guard<std::mutex> lock(m);
        if (data.empty())
            throw std::exception("empty");

        const std::shared_ptr<T> topElement(std::make_shared<T>(std::move(data.top())));
        data.pop();
        return topElement;
    }
    void pop(T& value) {
        std::lock_guard<std::mutex> lock(m);
        if (data.empty())
            throw std::exception("empty");

        value = std::move(data.top());
        data.pop();
    }
    bool empty() const {
        std::lock_guard<std::mutex> lock(m);
        return data.empty();
    }

    T top() const {
        std::lock_guard<std::mutex> lock(m);
        return data.top();
    }

private:
    std::stack<T> data;
    mutable std::mutex m;
};
```
- This `wrapper` class template does **not** introduce any **new** features or techniques
    * Rather, it simply applies the `mutex-related` techniques we have learned so far

### A Thread-Safe Queue
```cpp
#include <queue>
#include <memory>
template<typename T>
class QueueThreadSafe {
public:
    QueueThreadSafe() {}
    void push(T newValue) {
        std::shared_ptr<T> dataPtr(std::make_shared<T>(std::move(newValue)));      // construct here, not in pop()

        std::lock_guard<std::mutex> lock(mut);
        data.push(dataPtr);
        cv.notify_one();
    }

    void waitAndPop(T& value) {
        std::unique_lock<std::mutex> lock(mut);
        cv.wait(lock, [this] {return !data.empty(); });

        value = std::move(*data.front());
        data.pop();
    }
    std::shared_ptr<T> waitAndPop() {
        std::unique_lock<std::mutex> lock(mut);
        cv.wait(lock, [this] {return !data.empty(); });

        std::shared_ptr<T> frontElement = data.front();
        data.pop();
        return frontElement;
    }

    bool tryPop(T& value) {
        std::lock_guard<std::mutex> lock(mut);
        if (data.empty())
            return false;

        value = std::move(*data.front());
        data.pop();
        return true;
    }
    std::shared_ptr<T> tryPop() {
        std::lock_guard<std::mutex> lock(mut);
        if (data.empty())
            return std::shared_ptr<T>();

        std::shared_ptr<T> frontElement = data.front();         // copying shared_ptr is exception-safe
        data.pop();
        return frontElement;
    }
    bool empty() const {
        std::lock_guard<std::mutex> lock(mut);
        return data.empty();
    }

    T front() const {
        std::lock_guard<std::mutex> lock(mut);
        return *data.front();
    }
    T back() const {
        std::lock_guard<std::mutex> lock(mut);
        return *data.back();
    }
private:
    mutable std::mutex mut;
    std::queue<std::shared_ptr<T>> data;
    std::condition_variable cv;
};
```
- It's worth noting that `data` is a queue of `std::shared_ptr`s instead of just `T`
    * This is because of the **exception handling** 
    * Suppose that you use a queue of just `T`, and a `thread` waiting on `wait_and_try()` is **notified** but an **exception occurs**
        + for example, when a `std::shared_ptr` is **constructed**
    * In this case, `pop()` may **not** be properly **handled**
    * To address this issue, it is better to **move** the `construction phase` to the `push()` method
- This `wrapper` class template can be utilized in scenarios like GUI handling, as we discussed previously
    ```cpp
    template<typename T>
    void readQueue(QueueThreadSafe<T>& queue) {
        while (true) {
            std::shared_ptr<T> newData = queue.waitAndPop();
            std::cout << *newData << " is read" << std::endl;
        }
    }
    template<typename T>
    void writeQueue(QueueThreadSafe<T>& queue, const T& inputValue) {
        queue.push(inputValue);
    }

    int main() {
        QueueThreadSafe<int> queue;
        std::thread t1(readQueue<int>, std::ref(queue));
        std::thread t2(writeQueue<int>, std::ref(queue), 3);
        std::thread t3(writeQueue<int>, std::ref(queue), -1);

        t2.join();
        t3.join();
        t1.join();

        return 0;
    }
    /*
    possible outcome
    -1 is read
    3 is read
    */
    ```

## Design with Multiple Mutexes
The previous queue example was a `wrapper` class
- This time, we will implement a **real** `queue` data structure with **two `mutexes`**

### A Concurrent Queue
```cpp
#include <memory>
template<typename T>
class QueueThreadSafe
{
    struct Node {
        std::shared_ptr<T> data;
        std::unique_ptr<Node> next;
    };
public:
    QueueThreadSafe() : head(new Node), tail(head.get()) { }
    QueueThreadSafe(const QueueThreadSafe& other) = delete;
    QueueThreadSafe& operator=(const QueueThreadSafe& other) = delete;

    void push(T);

    std::shared_ptr<T> waitAndPop() {
        const std::unique_ptr<Node> headOld = waitPopHead();
        return headOld->data;
    }
    void waitAndPop(T& value) {
        const std::unique_ptr<Node> headOld = waitPopHead(value);
    }

    std::shared_ptr<T> tryPop() {
        std::unique_ptr<Node> headOld = tryPopHead();
        return headOld ? headOld->data : std::shared_ptr<T>();
    }
    bool tryPop(T& value) {
        const std::unique_ptr<Node> headOld = tryPopHead(value);
        return headOld;
    }

    bool empty() const {
        std::lock_guard<std::mutex> lockHe  (mutexHead);
        return isEmpty();
    }

    T front() const {
        std::lock_guard<std::mutex> lockHead(mutexHead);
        if (isEmpty())
            return T();
        return *head->data;
    }

private:
    Node* getTail() {
        std::lock_guard<std::mutex> lockTail(mutexTail);
        return tail;
    }
    bool isEmpty() {    // should be called after mutex is locked
        return (head.get() == getTail());
    }
    std::unique_ptr<Node> popHead() {
        std::unique_ptr<Node> headOld = std::move(head);
        head = std::move(headOld->next);
        return headOld;
    }

    std::unique_lock<std::mutex> waitForData() {
        std::unique_lock<std::mutex> lockHead(mutexHead);
        cv.wait(lockHead, [&] {return !isEmpty(); });
        return std::move(lockHead);
    }
    std::unique_ptr<Node> waitPopHead() {
        std::unique_lock<std::mutex> lockHead(waitForData());
        return popHead();
    }
    std::unique_ptr<Node> waitPopHead(T& value) {
        std::unique_lock<std::mutex> lockHead(waitForData());
        value = std::move(*head->data);
        return popHead();
    }

    std::unique_ptr<Node> tryPopHead() {
        std::lock_guard<std::mutex> lockHead(mutexHead);
        if (isEmpty())
            return std::unique_ptr<Node>();
        return popHead();
    }
    std::unique_ptr<Node> tryPopHead(T& value) {
        std::lock_guard<std::mutex> lockHead(mutexHead);
        if (isEmpty())
            return std::unique_ptr<Node>();
        value = std::move(*head->data);
        return popHead();
    }



    std::mutex mutexHead;
    std::unique_ptr<Node> head;
    std::mutex mutexTail;
    Node* tail;
    std::condition_variable cv;
};
template<typename T>
void QueueThreadSafe<T>::push(T newValue) {
    std::shared_ptr<T> newData(std::make_shared<T>(std::move(newValue)));
    std::unique_ptr<Node> tempNode(new Node);

    {
        std::lock_guard<std::mutex> lockTail(mutexTail);
        tail->data = newData;
        Node* const newTail = tempNode.get();
        tail->next = std::move(tempNode);
        tail = newTail;
    }
    cv.notify_one();
};
``` 
- This class manages two mutexes
    * `mutexHead` for `pop()` operations
    * `mutexTail` for `push()` operations
- This is the example use of this 
    ```cpp
    template <typename T>
    void populateData(QueueThreadSafe<T>& queue, const T &inputValue) {
        T data = inputValue;
        while (true) {
            std::this_thread::sleep_for(std::chrono::seconds(1));
            queue.push(data);
            data += data;
        }
    }
    template <typename T>
    void processData(QueueThreadSafe<T>& queue) {
        while (true) {
            auto newData = queue.waitAndPop();
            std::cout << *newData << " is processed" << std::endl;
        }
    }

    int main() {
        QueueThreadSafe<int> queue;
        std::thread threadRead(processData<int>, std::ref(queue));
        std::thread threadWrite(populateData<int>, std::ref(queue), 3);

        threadRead.join();
        threadWrite.join();

        return 0;
    }
    /*
    ongoing result
    3 is processed
    6 is processed
    12 is processed
    ...
    */
    ```

### A Concurrent List
Now, let's implement a **concurrent** `list` which supports **iterative operations**
```cpp
#include <memory>
template<typename T>
class ListThreadSafe {
    struct Node {
        std::mutex m;
        std::shared_ptr<T> data;
        std::unique_ptr<Node> next;
        Node() : next() {}
        Node(const T& value) : data(std::make_shared<T>(value)) { }
    };
public:
    ListThreadSafe() { }
    ~ListThreadSafe() {
        remove_if([](const Node&) {return true; });
    }
    ListThreadSafe(const ListThreadSafe& other) = delete;
    ListThreadSafe& operator=(const ListThreadSafe& other) = delete;


    void pushFront(const T& value) {
        std::unique_ptr<Node> newNode(new Node(value));
        std::lock_guard<std::mutex> lock(head.m);
        newNode->next = std::move(head.next);
        head.next = std::move(newNode);
        int a = 3;
    }
    template<typename Function>
    void forEach(Function operation) {
        Node* current = &head;
        std::unique_lock<std::mutex> lock(head.m);
        Node* nextNode = nullptr;
        while (nextNode = current->next.get()) {
            std::unique_lock<std::mutex> nextLock(nextNode->m);
            lock.unlock();
            operation(*nextNode->data);
            current = nextNode;
            lock = std::move(nextLock);
        }
    }
    template<typename Predicate>
    std::shared_ptr<T> findFirstIf(Predicate predicate) {
        Node* current = &head;
        std::unique_lock<std::mutex> lock(head.m);
        Node* nextNode = nullptr;
        while (nextNode = current->next.get()) {
            std::unique_lock<std::mutex> nextLock(nextNode->m);
            lock.unlock();
            if (predicate(*nextNode->data))
                return nextNode->data;
            current = nextNode;
            lock = std::move(nextLock);
        }
        return std::shared_ptr<T>();
    }

    template<typename Predicate>
    void remove_if(Predicate predicate) {
        Node * current = &head;
        std::unique_lock<std::mutex> lock(head.m);
        Node* nextNode = nullptr;
        while (nextNode = current->next.get()) {
            std::unique_lock<std::mutex> nextLock(nextNode->m);
            if (predicate(*nextNode->data)) {
                std::unique_ptr<Node> old_next = std::move(current->next);
                current->next = std::move(nextNode->next);
                nextLock.unlock();
            }
            else {
                lock.unlock();
                current = nextNode;
                lock = std::move(nextLock);
            }
        }
    }

private:
    Node head;
};
```
- The code below shows a raw usage example of this
    ```cpp
    template <typename T>
    void populateData(ListThreadSafe<T>& list, const T &inputValue) {
        T value = inputValue;
        for(unsigned count = 0, endCount = 7; count < endCount; count++) {
            list.pushFront(value);
            value += value;
        }
    }
    template <typename T>
    void printData(ListThreadSafe<T>& list) {
        std::cout << "populating..." << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(50));
        list.forEach([](const T& data) {std::cout << data << " "; });
        std::cout << std::endl;

        std::this_thread::sleep_for(std::chrono::seconds(3));
        std::cout << "after removing..." << std::endl;
        list.forEach([](const T& data) {std::cout << data << " "; });
    }
    template <typename T>
    void removeData(ListThreadSafe<T>& list, const T& targetValue) {
        std::this_thread::sleep_for(std::chrono::seconds(1));
        std::cout << "removing " << targetValue << std::endl;
        list.remove_if([&targetValue](const T& data) {return data == targetValue; });
    }


    int main() {
        ListThreadSafe<int> list;
        std::thread threadPopulate(populateData<int>, std::ref(list), 3);
        std::thread threadRemove(removeData<int>, std::ref(list), 6);
        std::thread threadPrint(printData<int>, std::ref(list));


        threadPopulate.join();
        threadRemove.join();
        threadPrint.join();

        return 0;
    }
    /*
    print result
    populating...
    192 96 48 24 12 6 3
    removing 6
    after removing...
    192 96 48 24 12 3
    */
    ```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}