2026-05-16 07:50
Status: #baby 
Tags: [[threading]]
# Potential issues with detaching a thread

Once you've created a `std::thread` you need to decide to allow the main thread to wait for it with a `join` or `detach` it. If don't decide before you program finishes running the thread with terminate with `std::terminate` meaning the program hard closes. You have to decide!

Detaching a thread puts that thread in state were the actual `std::thread` can be destroyed by it's destructor but the thread will continue to do stuff and stop on it's own. 

Additionally with detached threads you also need to make sure that all the data it's access is valid until it's finished. Be careful this is to do with life time objects, you have additional opportunities to mess this up with threading and detached threads.

```c++
#include <iostream>
#include <thread>

struct func 
{
	int& value;
	func(int& inValue) : value(inValue)
	{
	}

	void operator()() 
	{
		for (uint32_t i = 0; i < 10000000; ++i) 
		{
			//Here we have a potential access to a dangling reference.
			add(value);
		}
	}

	void add(int num) 
	{
		value += num;
	}
};

void bad()
{
	//IMPORTANT: localState is going to go out of scope BEFORE myThread can finish.
	int localState = 0;
	func myFunc{localState};
	std::thread t1{myFunc};
	t1.detach();
}

int main() 
{
	bad();
	//Meaning that here myThread might be running with a reference to localState that's now invalid memory.
	return 0;
}
```

This is a mistake in single threaded code as well but it might not show signs of something going wrong. This is all because we are still referencing `localState` causing UB. Read the comments to see why.

You can try and avoid this UB by copying all the shared data to function/object. Be very careful of pointers and references, especially if they are pointing to local variables. Unless you know for sure that the thread is going to finish before the function/scope ends.
# References
##### Main Notes
[[When are detached threads useful]]
[[Launching a thread]]
[[Basics of waiting for a thread]]
[[# Potential issues with joining a thread]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]