2026-05-16 08:00
Status: #baby 
Tags: [[threading]]
# Potential issues with joining a thread

You should know that you need to call `join` or `detach` before the life time of the `std::thread` ends. You can much pretty call `detach` as soon as the thread starts. However, you need to be careful were you wait for a thread with `join()` If you get an early return due to an error or an exception thing might not join and you get an`std::terminate()` will closing your program.

It's pretty simple but you need to be careful if you have branching code, you need to make sure that call `.join()` in both places.

```c++
#include <iostream>
#include <thread>

struct Work 
{
	Work(int inValue) : value(inValue)
	{
	}

	void operator()() 
	{
		for (uint32_t i = 0; i < 100000; ++i)
		{
			add(value);
		}
	}

	void add(int num) 
	{
		value += num;
	}

	int value;
};

int main()
{
	int localState = 1;
	Work work(localState);
	std::thread t1{work};
	try 
	{
		//Stuff happens here.
	}
	catch (...) 
	{
		t1.join();
		throw;
	}
	t1.join();
	return 0;
}
```

You guard against this with RAII if you put everything in the destructor.

```c++
class ThreadGuard
{
	public:
		explicit ThreadGuard(std::thread& inThread) : t(inThread)
		{}
		~ThreadGuard()
		{
			if(t.joinable())
			{
				t.join();
			}
		}
	private:
		std::thread& t;
		ThreadGuard(ThreadGuard const&) = delete;
		ThreadGuard& operator=(ThreadGuard const&) = delete;
};

struct Work 
{
	Work(int inValue) : value(inValue)
	{
	}

	void operator()() 
	{
		//Work gets done here.
	}

	int value;
};

void foo()
{
	int localState = 0;
	Work work(localState);
	std::thread t{work};
	ThreadGuard g{t};
}
```

The life time of the thread can be handled by setting the it as a reference to an object. As `ThreadGuard` instance `g` gets destroyed first when it falls out of the function foo() meaning that the destructor run.

In this example you have to pay attention to 2 thing.
1) The copy operator and copy constructor have been deleted, this is to make sure that `std::thread` doesn't outlive the scope by a copy operation.
2) We check in the destructor if the thread is `joinable()` this is to make sure we don't just twice.

If you don't need to wait for this thread to finish for a reason you don't have to worry about any of this, as it break the association between the thread with `std::thread` meaning that `std::terminate()` when `std::thread` goes out of scope.
# References
##### Main Notes
[[Basics of waiting for a thread]]
[[Potential issues with detaching a thread]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]