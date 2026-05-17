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

page 41