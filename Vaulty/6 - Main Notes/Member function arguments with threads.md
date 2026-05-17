2026-05-16 11:13
Status: #baby 
Tags: [[threading]]
# Member function arguments with threads

If you want to pass a member function that's public to `std::thread` also need to provide the object a suitable object pointer in the second argument.

```c++
struct Work
{
	void doWorkHere()
	{
	}
};
Work work;
std::thread t(&Work::doWorkHere, &work);
```

If you want to provide more arguments just pop them in after the object pointer.
# References
##### Main Notes
[[Non-const reference arguments with threads]]
[[Passing argument basics with threads]]
[[Arguments that can only be moved with threads]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]