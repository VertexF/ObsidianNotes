2026-09-19 11:34
Status: #baby 
Tags: [[OS]] [[threading]]
# Making a ticket spin lock

Since we have the hardware support for a [[What is fetch-and-add|fetch and add]] instruction we can use that to our advantage to create a ticketing system to allow threading to become fairer and avoid starvation unlike [[Making a spin lock with load-link and conditional store]] 

First need to show now that the lock has a `turn` that's global across all locks and a `ticket` variable.

```c++
struct Lock
{
	static int turn;
	int ticket;
};

void lockInit(Lock* lock)
{
	lock->ticket = 0;
	lock->turn = 0;
}
```

To acquire the lock we do a atomic [[What is fetch-and-add|fetch and add]] on the ticket variable, if the variable that's returns is not equal the currents lock turns we spin.

```c
Lock currentLock;
lockInit(&currentLock);
int fetchAndAdd(int *ptr)
{
	//The fetch
	int old = *ptr;
	//The add
	*ptr = old + 1;
	return old;
}

void lock(Lock* lock)
{
	int myTurn = fetchAndAdd(&lock->ticket);
	while(myTurn != lock->turn)
	{
		//Spin the thread.
	}
}
```

This requires us to add in a new variable into the lock itself that's global across all locks that increments by one. Each time this global `turn` variable get incremented by 1 it's the next threads turn to run. This allows the next thread to go because we are turning the previous.

When we unlock we just increment the turn variable by one.
```c++
void unlock(Lock* lock)
{
	lock->turn++;
}
```

Now on the next loop around the `lock->turn` in the while loop is a higher number so the next the `1 + ticket` thread goes next thanks for our atomic operation.  
# References
##### Main Notes
[[What is fetch-and-add]]
#### Source Notes
