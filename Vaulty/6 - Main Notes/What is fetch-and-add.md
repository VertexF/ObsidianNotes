2026-09-19 11:30
Status: #baby 
Tags: [[OS]] [[threading]]
# What is fetch-and-add

A fetch-and-add is an atomic operation that allows you to fetch something from memory and add `1` to it while return the old value back.

The C-style pseudo code might look something like this
```c
int fetchAndAdd(int *ptr)
{
	//The fetch
	int old = *ptr;
	//The add
	*ptr = old + 1;
	return old;
}
```

This can be used to create a [[Making a ticket spin lock|ticket lock|]]
# References
##### Main Notes
[[Making a ticket spin lock]]
#### Source Notes
