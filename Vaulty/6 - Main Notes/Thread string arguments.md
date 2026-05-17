2026-05-16 11:09
Status: #baby 
Tags: [[threading]]
# Thread string arguments

String a little more difficult to handle in this example

```c++
void foo(uint32_t num, const std::string& string)
{
	//Does stuff
}
std::thread t(foo, 3, "hello");
```

This is converted to `std::string` **ONLY** in the context of a new thread. This is important to note in this situation things can go wrong.

```c++
void foo(uint32_t num, cosnt std::string& string)
{
	//Does stuff
}
void bad(int value)
{
	char buffer[1024];
	sprintf(buffer, "%i", value);
	std::thread t(foo, 3, buffer);
	t.detach();
}
```

In this case the pointer, NOT the string that's passed to new thread. This means that `std::string` has to hunt down the actual character values, this may lead to the `std::string`  not completing the conversion before leaving the scope of the body, leading to UB.

The solution in my engines case and in the `std::string` case is to construct an object that converts the `char buffer[1024]` to an object before passing the value to the thread.

```c++
void foo(uint32_t num, cosnt StringBuffer& string)
{
	//Does stuff
}
void bad(int value)
{
	char buffer[1024];
	sprintf(buffer, "%i", value);
	
	StringBuffer string;
	string.init(1024, allocator);

	string.appendF("%s", buffer, value);
	
	std::thread t(foo, 3, buffer);
	t.detach();
	
	string.shutdown();
}
```

The reason you would need to do this is because the implicit conversion happening when you just pass the point happens too late because `std::thread` constructor copies the pointer without converting anything.
# References
##### Main Notes
[[Passing argument basics with threads]]
[[Non-const reference arguments with threads]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]
