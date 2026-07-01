2026-05-21 13:09
Status: #baby 
Tags: [[threading]]
# The problem with pointer and reference access shared data

This is complication that can happen if you pass in a pointer **INTO** a function that's dealing with shared data. Whether you using OOP or strict DOD it's bad. You need to consider how to you access data.

Not passing pointer and reference to protected data outside the scope of a lock. This can be by returned from the function, stored externally, making the memory visible, or by pass non-const  reference/pointer in them by arguments.

```c++
struct Data
{
	int a;
	StringView name;
	
	void doWork();
};
struct Buffer
{
	Data data;
	std::mutex mutex;
	
	void allocateData(BaseAllocator* allocator)
	{
		std::lock_guard<std::mutex> lock(mutex);
		allocator->(data);
	}
};
Data *unprotecedData;
Buffer buffer;
void foo()
{
	HeapAllocator heapAllocator;
	//This vv below is the problem child we have given a pointer to a piece of data that's meant to be shared between thread and need to be protected.
	unprotecedData = &buffer.data;
	//Meaning here it gets locked by a mutex.
	buffer.allocateData(heapAllocator);
	//Then our work is undone because we do something on that data we were meant to be protected EVEN though we locked it earlier.
	unprotecedData->doWork();
}
```
# References
##### Main Notes
[[Introduction to mutex and locking]]
#### Source Notes
[[C++ Concurrency - Protected Shared Data]]