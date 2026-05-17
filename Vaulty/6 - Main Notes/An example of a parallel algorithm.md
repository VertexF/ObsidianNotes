2026-05-16 15:35
Status: #baby 
Tags: [[threading]]
# An example of a parallel algorithm

This algorithm below is an example of how threads can be used to make something parallel. We are making a parallel version of std::accumulate which simply adds all the values in a standard container.

Don't worry too much if you're coming back to this after a break there should be more notes on how to write parallel algorithms which will fill in stuff I decided to leave out.

```c++
#include <thread>
#include <numeric>
#include <vector>
#include <iostream>

template<typename iter, typename T>
struct AccumulateBlock
{
	void operator()(iter first, iter last, T& result)
	{
		result = std::accumulate(first, last, result);
	}
};

template<typename iter, typename T>
T paraAccumulate(iter first, iter last, T init)
{
	unsigned long const length = std::distance(first, last);
	if(!length)
	{
		return init;
	}

	unsigned long const minPerThread = 25;
	unsigned long const maxThreads = (length + minPerThread - 1) / minPerThread;
	unsigned long const hardwareThreads = std::thread::hardware_concurrency();
	unsigned long const numThreads = std::min(hardwareThreads != 0 ? hardwareThreads : 2, maxThreads);
	unsigned long const blockSize = length / numThreads;
	std::vector<T> results(numThreads);
	std::vector<std::thread> threads(numThreads - 1);
	iter blockStart = first;
	for(unsigned long i = 0; i < (numThreads - 1); ++i)
	{
		iter blockEnd = blockStart;
		std::advance(blockEnd, blockSize);
		threads[i] = std::thread
		(
			AccumulateBlock<iter, T>(),
			blockStart,
			blockEnd,
			std::ref(results[i])
		);
		blockStart = blockEnd;
	}
	AccumulateBlock<iter, T>()(blockStart, last, results[numThreads - 1]);

	for(auto& entry : threads)
	{
		entry.join();
	}

	return std::accumulate(results.begin(), results.end(), init);
}

int main()
{
	std::vector<int> list{33, 494, 129, 298, 839, 209, 209, 584, 930};
	int result = paraAccumulate(list.begin(), list.end(), 0);
	std::cout << result << std::endl;

	return 0;
}
```

The first thing we are doing in our `paraAccumulate` function is returning early if the container is empty. We can then divide the number of element by the minimum block in order to get the maximum number of threads. This avoid creating too many threads with a small list of 5 or something. This is to avoid **oversubscription** of threads.

If you get 0 from `std::thread::hardware_concurrency()` you should make sure you pick a number that isn't too low to not take advantage of thread or too high to slow down single threaded hardware.

Note because of [[Containers with threads]] we can solve everything in a container. Once we have figured out a good number of threads to work with. 
# References
##### Main Notes
[[Choosing the number of threads at runtime]]
[[Containers with threads]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]