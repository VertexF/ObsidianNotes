2026-02-11 11:59
Status: #baby 
Tags: [[performance]] [[DOD]]
# Reducing struct memory alignment

When dealing with memory alignment you need to remember that the largest member variable makes the memory alignment. So if you can reduce the biggest member to a smaller type here are two examples

```c++
struct A 
{
	//uint64_t id;
	uint32_t id;
	uint32_t health;
	bool isDead;
};

struct B
{
	//A *apples;
	uint32_t applesHandle;
	uint32_t amountOfApples;
};
```

This `id` can be **uint32_t** that would only be 4 bytes aligned struct rather than 8 potientially helping how much memory you can pull from memory. If possible you might be able to remove pointers from struct because they are 8 bytes in size to use handles which are 4. Be careful though you lose type safety when you do that.
# References
##### Main Notes
[[Aligning your structs for better performances]]
[[Storing bools out of bounds]]
[[Moving data to helps alignment]]
#### Source Notes
[[Intro to Data Oriented Design for Games]]