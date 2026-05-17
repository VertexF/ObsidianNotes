2026-05-16 14:33
Status: #baby 
Tags: [[threading]]
# Moving threads with functions

The movement of `std::thread` allows you to be able to move threads in to functions as arguments or return them out of functions.

```c++
std::thread foo()
{
	void someFunction();
	return std::thread(someFunction);
}
std::thread bar()
{
	void someOtherFunction(int);
	std::thread t(someOtherFunction, 42);
	return t;
}
```

Both of these are valid moves.

You can move `std::thread` in as arguments too.

```c++
void foo(std::thread t){}
void bar()
{
	void someFunction();
	//Approach 1
	foo(std::thread(someFunction));
	//Approach 2
	std::thread t(someFunction);
	foo(std::move(t));
}
```
# References
##### Main Notes
[[Errors that can happen with moving threads]]
[[Using moves to safe guard against thread life time mistakes]]
[[Containers with threads]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]
