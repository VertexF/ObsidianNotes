2026-09-19 19:08
Status: #baby 
Tags: [[OS]] [[threading]]
# Making a block lock with a queue

A spin always spins threads while they wait, if a context switch happens at the wrong time this can cause threads to spin for the entire CPU time slice. A blocking thread buts threads to sleep instead of letting them spin.

To allow these to work we need a queue of threads that have been put to sleep. We add a thread to the queue if they try to acquire a lock and get removed from the queue and woken up after a unlock has been called.

There isn't hardware software for sleeping and waking threads so you'll need to rely on the operating system. With this in mind we can use the simplest spin lock the [[Making a test-and-set spin lock|test-and-set spin lock]] and create a blocking lock out of it, this is for more efficiency.

Here the Solaris OS version of sleep and waking threads with `park()` and `unpark(threadID)` we wake the the thread with an ID. The `compareAndExchange` is explained here [[What is swap exchange]]

```c++
struct Lock
{
	int flag; //Used to check if the lock is aquired.
	int guard;
	Queue* queue; //This is just a pointer to a basic queue.
};

void lockInit(Lock* lock)
{
	lock->flag = 0;
	lock->guard = 0;
	queueInit(lock->queue);
}

void lock(Lock* lock)
{
	while(compareAndExchange((&lock->guard, 1) == 1)
	{
		//Aquire guard lock by spinning.
	}
	
	if(lock->flag == 0)
	{
		lock->flag = 1;
		lock->guard = 0;
	}
	else
	{
		queueAdd(lock->queue, getCurrentThreadID());
		setpark(); //A conditional park, the conditional check for an unlock if that happens if does sleep the thread.
		lock->guard = 0;
	}
}

void unlock(Lock* lock)
{
	while(compareAndExchange((&lock->guard, 1) == 1)
	{
		//Aquire guard lock by spinning.
	}
	
	if(queueEmpty(lock->queue))
	{
		//No thread wants this lock.
		lock->flag = 0;
	}
	else
	{
		unpark(queueRemove(lock->queue));
	}
	
	lock->guard = 0;
}
```

We only end up spinning a little bit when an unlock happening or when another thread is racing to try and get the lock for itself.

We use this guard variable to avoid race condition. We have to reset them all to `0` before we do anything or a lock could avoid the spinning protection and go on reek havoc.

`setpark();` Is a conditional sleep, the conditional check for an unlock if that happens if does sleep the thread. 

```c++
void badLock(Lock* lock)
{
	while(compareAndExchange((&lock->guard, 1) == 1)
	{
		//Aquire guard lock by spinning.
	}
	
	if(lock->flag == 0)
	{
		lock->flag = 1;
		lock->guard = 0;
	}
	else
	{
		//Bad and origianl implemnetion here
		queueAdd(lock->queue, getCurrentThreadID());
		lock->guard = 0;
		park();
	}
}
```

The reason you would want this if that's it's possible with a context switch that happen just as `pack()` get called, then a context switch to the a thread that calls `unlock()` happen. This causes a race condition because the thread should be woken when an unlock is called, but if we put a thread to sleep after.

Generally speaking this type of lock is still in use today but the details are different the techniques are the same.
# References
##### Main Notes
[[What is swap exchange]]
[[Making a test-and-set spin lock]]
#### Source Notes
