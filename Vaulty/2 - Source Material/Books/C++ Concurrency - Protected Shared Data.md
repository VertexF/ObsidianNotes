# Reference C++ConcurrencyinAction

##### An introduction to the mutex
Shared data that we are concerned with is anything that has an **invariant** like pointers in a linked list that can be broken at run time, causing a bad race condition.

A mutex (**mut**ual **ex**clusion) is a primitive that tells threads that to wait before accessing shared data if another thread is accessing it. You **lock** the mutex when a thread access shared data, and you on **unlock** it once a thread has finished working on it. 

The standard library make sure that when a thread tries to lock a thread it can't, only when it's **unlocked** is that possible. This make sure that threads have self-consistent view on shared data.

You can't just 100% rely on these, you need to structure your interface to your code to avoid race conditions as well. You have to be careful you don't end up with a **deadlock** and protecting too much or too little.
##### A basic use of a mutex
In C++ you need to use the `std::mutex` instance. You lock a mutex with `lock()` member function and unlock it with `unlock()` member function. This can be tricky to get right so using RAII with mutex version of the mutex to help with object life time and things like `std::expections`.

You can use something called `std::lock_guard` which is a class template for RAII which locks itself on construction and unlocks itself unlocks destruction. It takes in a mutex as a template argument.

```c++
#include <list>
#include <mutex>
#include <algorithm>
std::list<int> someList;
std::mutex someMutex;
void addToList(int newValue)
{
	std::lock_guard<std::mutex> guard(someMutex);
	someList.push_back(newValue);
}
bool listContains(int valueToFind)
{
	std::lock_guard<std::mutex> guard(someMutex);
	return std::find(someList.begin(), someList.end(), valueToFind) != someList.end();
}
```

##### The problem with pointer and reference access shared data
This is complication that can happen if you pass in a pointer **INTO** a function that's dealing with shared data. Whather you using OOP or strict DOD it's bad. You need to consider how you access data.

Not passing pointer and reference to protected data outside the scope of a lock. This can be by return from the function, storing them externally visible memory, or by passint them by arguments.

```c++
struct Data
{
	int a;
	StringView name;
	
	void doWork();
};
struct Buffer
{
	Data data;
	std::mutex mutex;
	
	void allocateData(BaseAllocator* allocator)
	{
		std::lock_guard<std::mutex> lock(mutex);
		allocator->(data);
	}
};
Data *unprotecedData;
Buffer buffer;
void foo()
{
	HeapAllocator heapAllocator;
	//This vv below is the problem child we have given a pointer to a piece of data that's meant to be shared between thread and need to be protected.
	unprotecedData = &buffer.data;
	//Meaning here it gets locked by a mutex.
	buffer.allocateData(heapAllocator);
	//Then our work is undone because we do something on that data we were meant to be protected EVEN though we locked it earlier.
	unprotecedData->doWork();
}
```

##### Spotting race conditions in interfaces.
Race condition can happen even if you protect shared data. If you have a doubly linked list, you need to make sure that all 3 nodes are protected when deleting. As the pointers going to and from the deleting nodes are also related to deletion process. The general idea here is just because individual operations are safe doesn't mean the entire thing is thread safe.

For example with a stack you can have a top() and pop() operation that happens that returns a copy rather a reference which will protect data, but you'll still run into a race condition. Even if you use mutexes in your operations. In our example, empty() and size() can't be relied on.

If the object aren't shared then your chill.

```c++
std::stack<int> stack;
if(stack.empty() == false)
{
	int const value = stack.top();
	stack.pop();
	doSomething(value);
}
```

This is safe in single threaded enviroment code. It's UB to call `top()` on an `empty()` stack.  What you could do is throw an expection, but now `empty()` is slower to run. What you need to do is make sure that it's not possible to use `top()` in any thread if it's empty by calling `empty()`.

How to fix all this is that you have a `std::mutex` that only allows internal member functions one thread at a time, allowing everything to be interleaved, then `doSomething(value);` can do thing concurrently. This allows the same code to run in different threads with different data.

You'll get this example below

| Thread A             | Thread B             |
| -------------------- | -------------------- |
| if(stack.empty())    | ----                 |
| ----                 | if(stack.empty())    |
| int v = stack.top(); | ----                 |
| ----                 | int v = stack.top(); |
| stack.pop();         | ----                 |
| -----                | stack.pop();         |
| doSomething(v);      | doSomething(v);      |
Do you see the problem here? We have a double pop! So we get the same value twice in the `top()` and `pop()` to values. This is much harder to find, since you wont get UB or a crash.

How do you solve this? Well you need to do `top()` and `pop()` together at the same time. Expections can still have things go wrong with a copy-constructor expection.