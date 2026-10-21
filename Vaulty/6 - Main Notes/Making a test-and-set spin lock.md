2026-09-19 17:18
Status: #baby 
Tags: [[OS]] [[threading]]
# Making a test-and-set spin lock

To build a spin lock with this you would need to spin on the compare and exchange and wait for a true value to be return then continue on to a critical section.

```c
lock_t lock;
void lock(lock_t* lock)
{
	while(compareAndExchange(&lock->flag, 0, 1) == 1)
	{
		// Thread spins here.
	}
}

lock(lock->flag);
//Critical section here.
```

What we are saying here is if the flag value and the old value is equal to `0` we set the value to `1` and return true. Else we just loop until the flag and the value at that point is equal to `0`.

This allows the one thread to wait for another unlock and set that piece of memory to `0`.
# References
##### Main Notes
[[What is swap exchange]]
#### Source Notes
