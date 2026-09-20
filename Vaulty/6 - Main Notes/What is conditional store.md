2026-09-19 08:43
Status: #baby 
Tags: [[OS]] [[threading]]
# What is conditional store

It takes a condition and only if that condition is true will it update the value. It's similar to [[What is swap exchange|swap exchange]] but it doesn't return a value.

```c
bool storeConditional(int *ptr, int value)
{
	if(<no updates has happened to the registers since the load-link fetch>)
	{
		*ptr = value;
		return true;
	}
	else
	{
		return false;
	}
}
```

What is the `<no updates has happened to the registers since the load-link fetch>` this instruction is very coupled with a load-link instruction, it's almost 1 instruction.

This is used to make a lock for threading with [[What is load-linked|load-link]] 
# References
##### Main Notes
[[What is load-linked]]
[[What is swap exchange]]
[[Making a spin lock with load-link and conditional store]]
#### Source Notes
