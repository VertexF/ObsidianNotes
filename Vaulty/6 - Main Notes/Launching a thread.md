2026-05-16 07:47
Status: #baby 
Tags: [[threading]]
# Launching a thread

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
# References
##### Main Notes
[[Potential issues with detaching a thread]]
[[Basics of waiting for a thread]]
[[Introduction to multithreading]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]