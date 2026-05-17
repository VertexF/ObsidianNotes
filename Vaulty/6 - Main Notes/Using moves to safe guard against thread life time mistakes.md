2026-05-16 14:36
Status: #baby 
Tags: [[threading]]
# Using moves to safe guard against thread life time mistakes

With `std::move` being supported you can build a ScopeThread class that we force threads to be joined or detached the ScopeThread class does out of scope.

This guards against threads leaving the scope of something without being joined or detached.

```c++
class ScopeThread
{
	public:
		explicit ScopeThread(std::thread inThread) : t(std::move(inThread)) 
		{
			if(t.joinable() == false)
			{
				throw std::logic_error("No thread or thread is not joinable");
			}
		}
		~ScopedThread()
		{
			t.join();
		}
		ScopedThread(const ScopedThread& ) = delete;
		ScopedThread operation=(const ScopedThread& ) = delete;
	private:
		std::thread t;
};
struct Work
{
	void operator()(int num){ doWork(num); }
	void doWork(int num){ /*Something here*/ }
	int data;
};
void foo()
{
	int localState = 0;
	Work work;
	ScopedThread thread{ std::thread( work(localState)) };
	//Continue work in the current thread here.
}
```

We are just creating the thread on top of the `ScopeThread` Nothing really makes this special apart from the fact we make sure that the thread we are passing into the class hasn't already been joined.
# References
##### Main Notes
[[Moving threads with functions]]
[[Errors that can happen with moving threads]]
[[Containers with threads]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]