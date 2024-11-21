---
title: "C++ Concurrency : Threads Management"

categories:
    - cpp

tags:
    - [C++, Programming Language, Concurrency, Threads Management, thread]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-11-20
---

# C++ Concurrency : Chapter 2

> 이 포스트는 `C++ Concurrency in Action (2nd ed.)`을 바탕으로 작성되었습니다.

## `<thread>`
In order to use **functions** and **classes** for managing `threads`, you need to include `<thread>` header file

every cpp program -> at least one thread -> runing main() -> after this thread has been launchend -> you can then luanch additional threads with their own initial entry point function
-> thread exits when the entry point functino returns

## construction of `std::thread`
to start a thread, you need to contruct `std::thread` object with the initial point function (it can be any functor)
creating a std::thread does immediately start the function (or callable) in a separate thread. 

as i mentioned above, the thread is terminated when its functor returns
-> what happens if you don't speicfy any initilal functor -> then it regards as the initla functor returns immediately -> therefoer -> the default constructor would create a thread -> however, that thread will be destroyed right after it is created because it doesn't have any inital functor to call and this is regarded as the virtual functor returns immediately although it deosn't exist
and you cannot speicfy the initial functor to the terminated thread which means you can do nothing with the terminated thread
hence, if you want to run a functor in a separte thread, you need to speicfy it when `std::thread` is constructed

and initial entry functor is often referred as thread function



when you pass a function object as the initial functor, if you pass the temporary object to, the compiler will interpret it as function declaration not object construction
-> in order to fix this issue, you need to add additonal parathenses or use braced (uniform) initilaization
```c++
std::thread myThread(BackgroundTask());     // won't start
// now start
std::thread myThread((BackgroundTask()));   
std::thread myThread{BackgroundTask()};    
```
it's worth noting that lambda expression returns the tempoarary unanmed function object, but you can use it normally, and it works
```c++
std::thread myThread([] { /* do something */ });    // start as well
```

once the thread has been initiated, you need to **explicitly** decidie whether to wait for it to finish (call `.join()`) or to leave it to run on its own (call `.detach()`) before the `std::thread` object is destroyed
-> otherwise, the destructor calls `std::terminate`
the point of making decision is about the destruction of the `std::thread` not the end of the initial functor
    -> which means that it's totally okay to make decision after the initial functor returns if the `std::thread` is not destroyed yet
if you call `.detach()` you need to ensure that the data accessed by the thread is **valid**
    -> because accessing an object which has been deroyted is undefined
    -> detached threads are often called `daemon threads` -> such threads are typically long-running    
it's worth noting that the code after calling `.join()` cannot do anything with the `std::thread` object because `.join()` waits until the intial functor returns
    -> which means that if the control flow reads the code below `.join()`, this means that the `std::thread` is terminated


### exception handling
the place where you call `.detach()` is not that important -> hence usually it's called right after it's launched
but when it comes to `.join()`, you need a deep consideration regarding its proper place to put
there mgith a situation where call to `.join()` is skipped because of the exceptions
-> in order to handle this, you can use the concept of `RAII`
```c++
class ThreadGuard {
public:
    explicit ThreadGuard(std::thread &inputT) : t(inputT) {}
    ~ThreadGuard() {
        if(t.joinable())
            t.join();
    }

    // ThreadGuard is constructed only with the reference to existing thread
    ThreadGuard(const ThreadGuard&) = delete;
    ThreadGuard& operator= (const ThreadGuard&) = delete;
private:
    std::thread& t;
};
```
- the point of this method is to call `.join` when the destructor of `ThreadGuard` is called
    * because it's called eariler than the actual `std::thread` object because it's constructed after the `std::thread` is constructed 
- `joinable()` is important
    * because if you call `join()` more than once, it will result in undefined behavior, which is typically a runtime error because first call of `join` terminate the thread


## Argument passing
you can easily pass the arguments to the initila functor by sending them after inition functor to the constructor of `std::thread`

argument passing occurs twice -> 1: from the original code to contructor of `std::thread` -> 2: from constructor to initial functor
if we pass rvalue, it works as we expected (becasue the constructor of `std::thread` supports perfect forwarding)

however, when it comes to lvalue
2nd passing works as we expected
1st passing slightly different -> copy: works properly -> reference: copies it -> how to fix it -> use `std::ref` utility function so that 1st passing works as we anticipated
```c++
struct MyClass {
	MyClass() { std::cout << "worked" << std::endl; }
	MyClass(const MyClass &a) { std::cout << "worked" << std::endl; }
};

void f(MyClass a) {
	std::cout << "hello" << std::endl;
}

int main() {
	MyClass a;
	std::thread myThread(f, a);
	ThreadGuard g(myThread);

	return 0;
}
worked
worked
worked
hello
```
```c++
struct MyClass {
	MyClass() { std::cout << "worked" << std::endl; }
	MyClass(const MyClass &a) { std::cout << "worked" << std::endl; }
};

void f(const MyClass &a) {
	std::cout << "hello" << std::endl;
}

int main() {
	MyClass a;
	std::thread myThread(f, a);
	ThreadGuard g(myThread);

	return 0;
}
worked
worked
hello
```
```c++
struct MyClass {
	MyClass() { std::cout << "worked" << std::endl; }
	MyClass(const MyClass &a) { std::cout << "worked" << std::endl; }
};

void f(MyClass &a) {
	std::cout << "hello" << std::endl;
}

int main() {
	MyClass a;
	std::thread myThread(f, std::ref(a));
	ThreadGuard g(myThread);

	return 0;
}
worked
hello
```

detach -> special caution regarding passing handle to local object -> it might be destroyed but accessed by the separate thread


## Transferring ownership
`std::move()` is used to transfer ownership
```c++
std::thread t1(f);
std::thread t2 = std::move(t1);
```
- once the onwership has been transferred, the orignal thread object no longer needs to call `.detach()` or `.join()` (technically speaking, it cannot) because it has no thread to manage

it's possible for moved-from `std::thread` object to get the ownership from the running thread
```c++
std::thread t1(f);
std::thread t2 = std::move(t1);
t1 = std::move(t2);
```
it's also possible for terminated `std::thread` object to get the ownership from the running thread
```c++
std::thread t1(f);
std::thread t2;
t2 = std::move(t1);
```

it's worth noting that if you move to the `std::thread` object which already has a thread to manage, `std::terminate()` is called
```c++
std::thread t1(f1);
std::thread t2(f2);
t2 = std::move(t1);     // terminated
```

### upgrade ThreadGuard
```c++
class ThreadGuard {
public:
	explicit ThreadGuard(std::thread&& inputT) : t(std::move(inputT)) {}
	~ThreadGuard() {
		if (t.joinable())
			t.join();
	}
	// you can add other interfaces if you want
	void join() {
		if (t.joinable())
			t.join();
	}

	// ThreadGuard is constructed only with the reference to existing thread
	ThreadGuard(const ThreadGuard&) = delete;
	ThreadGuard& operator= (const ThreadGuard&) = delete;
private:
	std::thread t;
};
```

## Extra Details

### Hardware_concurrency
std::thread::ahrdward_concurrewncy() returns the number of threasda taht can truely run concurrently for a giaven execution of a program
it can be used to decide the maximum number of threads to use between the limination you set or the actually availble number
```c++
constexpr int MY_EXPECTATION_MAX = 24;
int realMaximumNumber = std::min(std::thread::hardward_concurrency(), MY_EXPECTATION_MAX);
realMaximumNumber--;    // becasue one thread is managing main()
std::vector<std::thread> threads(realMaximumNumber);
for(int i =0; i < realMaximumNumber; i++) {
    threads[i] = std::thread(f, args);
}
for(auto & thread : threads)
    thread.join();
```
- if you are tyring to use more threads than the available one, performance decreases (this is called oversubscription)

### get_id
if std::thread dosn't have an associated thread of execution, it returns a default-constructed std::thrad::id object which means "not any thread"

each thread has its own id
there are two ways to obtain it
1-> std::thread object -> .get_id() method
2 -> std::this_thread::get_id() function -> returns hte id for the current thread

you can use this id object as the key for ordered or unordered associative container

using get_id you can identify the thread object so that you can perform a speical job for a specific thread




[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}