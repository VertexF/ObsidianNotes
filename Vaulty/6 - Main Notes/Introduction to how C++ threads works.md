2026-05-15 09:47
Status: #baby 
Tags: [[threading]]
# Introduction to how C++ threads works

When writing concurrent code it's basically the same as non-concurrent code, however functions will be running concurrently to the rest of the application. We just have to make sure shared resource between threads are safely accessed, this includes memory and CPU.

Here we have hello world
```c++
#include <iostream>

int main() 
{
	std::cout << "Hello world!" << std::endl;

	return 0;
}
```

Here we have concurrent hello world.
```c++
#include <iostream>
#include <thread>

void print() 
{
	std::cout << "Hello world!" << std::endl;
}

int main() 
{
	std::thread t1(print);
	t1.join();
	return 0;
}
```

Thing for handling threads can be found in the `#include <thread>` where as stuff for handling  shared data can be found in a different header.

We had to move out the printing code into something called an **initial function** these are taken into thread object via the constructor. The main threads or the initial thread **initial function** is literally `int main()`. For every other thread we create we create it with an **initial function** like we did with like `std::thread t1(print);`

The main thread starts at `main()` our created thread starts at `print()` if we don't pause our main thread to wait for second thread it's possible we will reach the end of `main()` before `print()` has a chance to run. That's what the `t1.join();` is for it's telling our main thread to wait until our `t1` has finished printing to standard out has happened in the second thread.

If the main thread has nothing to do you've planned things wrong. For this application, it's not worth threading because the main thread is doing nothing. Be aware of this.
# References
##### Main Notes
[[Introduction to multithreading]]
#### Source Notes
[[C++ Concurrency - Hello, world of concurrency in C++]]