# Reference C++ConcurrencyinAction

##### Lauching a thread
If you want to use objects say like a physics struct and have that launch a thread you need to us the () operator `operation()` this allows C++ to treat your object as if it's a function.

```c++
struct Work 
{
	void operator()() 
	{
		add();
		add2();
		std::cout << value << std::endl;
	}

	void add() 
	{
		value = 2 * 1000;
	}

	void add2() 
	{
		value -= 50;
	}

	int value;
};

int main() 
{
	Work work;

	std::thread t1(work);
	t1.join();
	return 0;
}
```

This is because when you type the code `work();` in this case it will run whatever is in the `operator()()` function in our struct. Note you can't do this with thread objects, `std::thread t(work.add);` the compiler would know what you mean by this.
###### To wait or not
Once you've created a `std::thread` you need to decide to allow the main thread to wait for it with a `join` or detach it. If don't decide before you program finishes running the thread with terminate with `std::terminate`. You have to decide!

Joining the thread is still even necessary even if the thread has finished with it. Detacting a thread put that thread in state were the actual `std::thread` can be destroyed by it's constructor but it will continue to do stuff and stop on it's own. Addtionally with detached threads you also need to make sure that all the data it's access is valid until it's finished. Be careful this is to do with life time objects, you have additional oppunities to mess this up with threading and deteched threads.

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

This is a mistake in single threaded code as well however it might not show signs of something going wrong. This is all because we are still referencing `localState` causing UB.

You can try and avoid this UB by making all by copying all the shared data to function/object. Be very careful of pointer and reference, especially if they are pointing to local variables. Unless you know for sure that the thread is going to finish before the function/scope ends.
##### Basics of waiting on threads
You if you need to wait for a `std::thread` to finish you just the `.join` function call on the thread object. This is a heavy weight synchronisation function call, as it waits for the entire thread to finish. If you want finer control you need **conditional variable** and **futures**.

Calling `join()` also cleans up any storage that's tied to that thread. Meaning you can't call `join()` again and `joinable()` will return false.
##### Joining a thread when expection happen.
You should know that you need to call `join` or `detech` to the life time of the `std::thread` ends. You an pretty call `detech` as soon as the thread starts. However, you need to be careful were you wait for a thread with `join()` If you get an early return due to an error or an expection like the window resizing and now you've returned early in your render loop, and you don't join, `std::terminate()` will close your program.

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