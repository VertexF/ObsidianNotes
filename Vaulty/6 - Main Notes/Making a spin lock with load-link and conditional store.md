2026-09-19 08:48
Status: #baby 
Tags: [[OS]] [[threading]]
# Making a spin lock with load-link and conditional store

To make a better lock that a [[What is swap exchange|swap exchange]] you'll want to use both [[What is conditional store|conditional store]] 
and [[What is load-linked|load-linked]], as the swap exchange will not detect if the value has value has been restored. So lets break it down.

Lets bring in both load-link and conditional store c-style code to see how we can build the lock. Remember all this code is pseudo like C code and not actually real.
```c
bool storeConditional(int *ptr, int value)
{
	if(<no updates have occurred to that location since the load-link>)
	{
		*ptr = value;
		return true;
	}
	else
	{
		return false;
	}
}

void* loadLink(int ptrIndex)
{
	return mem[ptrIndex]; //mem[] is just a pretend all physical RAM
}
```

With this we can build up our lock.

```c
void lock(lock_t* lock)
{
	while(1)
	{
		while(loadLinked(&lock->flag) == 1)
		{
			//Spin into 0.
		}
		if(storeConditional(&lock->flag, 1) == 1)
		{
			//If the set to 1 was a success: done
			//Otherwise: try again; 
			return;
		}
	}
}
```

Here `while(loadLinked(&lock->flag) == 1)` we are rechecking a flag value over and over again to see if it's set to `0` If it use we move on. 

Then we hit `if(storeConditional(&lock->flag, 1) == 1)` since we loaded into a register with load-linked the condition within `storeConditionl` get triggered. If we are attempting to write to this register and it **HASN'T** been written to between the load-link and the store conditional we update that flag condition with `1` and return true, meaning we can leave this lock function and go to the critical section of the code. 

What's cool about this extra check is if 2 threads both have the load-linked value loaded in their registers, one thread will fail and go back to spinning because of the store conditional. Seeing an update has happened to that value since a load-link and will fail to store and return false.
# References
##### Main Notes
[[What is swap exchange]]
[[What is conditional store]]
[[What is load-linked]]
#### Source Notes
