2026-05-16 15:58
Status: #baby 
Tags: [[threading]]
# Identifying threads

Each `std::thread` has a `std::thread::id` that tells you the ID. You can get this in 2 ways.

One is by using the `get_id()` function from the associated `std::thread` object. If the `std::thread` doesn't have an associated thread of execution and you call `get_id()` it will return the default constructed "not any thread".

Two you can use the `std::this_thread::get_id()` which tells you what the ID is of the current thread.

You can compare and copy these guys. If they are both the same or "not a thread" they are the same thread. As you have all the operators with the ID this can be used as a key in maps and other things, you and also `std::hash<std::thread::id>` which can be very helpful.

If you have a std::thread::id you can check if you want that type of thread to a certain task or not. For example

```c++
std::thread::id masterThread;
void someCorePartOfAlgorithm()
{
	if(std::this_thread::get_id() == masterThread)
	{
		doMasterThreadWork();
	}
	doCommonWork();
}
```

# References
##### Main Notes

#### Source Notes
[[C++ Concurrency - Managing Threads]]