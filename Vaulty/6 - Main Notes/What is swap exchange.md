2026-09-19 08:23
Status: #baby 
Tags: [[OS]] [[assembly]] [[threading]]
# What is swap exchange

This is a x86 instruction that can be used for threading. It uses hardware built into the CPU to check if an expected value is there, if an expect value is there it exchanges it for another value. The x86 instruction is called `xchg`

This is an example of what a c-style implementation in hardware might look like.
```c
bool compareAndExchange(void* ptr, int *oldValue, int *newValue)
{
    if(*ptr != *old)
        return false
	else
		*ptr = *newValue;

    return true
}
```
# References
##### Main Notes
[[Making a test-and-set spin lock]]
#### Source Notes
