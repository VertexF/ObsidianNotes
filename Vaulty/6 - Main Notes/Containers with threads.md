2026-05-16 14:40
Status: #baby 
Tags: [[threading]]
# Containers with threads

This is pretty straight forward but you need to make sure that your containers you write are move aware and will move instead of copy. Something like a std::vector does that for us.

```c++
void doWork(unsigned int id){}
void foo()
{
	std::vector<std::thread> threads;
	for(uint32_t i = 0; i < 20; ++i)
	{
		threads.emplace_back(doWork, i);
	}
	for(auto& entry : threads)
	{
		entry.join();
	}
}
```

This might be what required if you have an algorithm that has a bunch of threads that need to all finish after doing something.

Note we are assuming that the `doWork()` function has work that each thread can do independently. You have to do more work if the returning values affect memory that the threads depend on. 
# References
##### Main Notes
[[The standard thread ownership rule]]
[[Errors that can happen with moving threads]]
[[Moving threads with functions]]
[[Using moves to safe guard against thread life time mistakes]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]