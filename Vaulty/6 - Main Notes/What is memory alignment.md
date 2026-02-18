2026-02-11 10:08
Status: #baby 
Tags: [[performance]] [[DOD]]
# What is memory alignment

When we talk about alignement of a struct we are saying that everything is aligned in memory by the biggest thing in the struct. 

When storing memory you can't just store 1 bit in memory by itself it needs to be nicely aligned with other data. Meaning that things get padded by the alignment.

So the string here is 8 bytes container.

What you would want to is to do what we did with [[Entity component system bad]] and turn those structs into an array

```c++
enum State : uint8_t
{
	DEAD,
	FREEZY_ALIVE,
	ALIVE,
}

struct Enemy
{
	std::string id;
	State state;
};
```

Finally you would want to remove that string from the struct and use a int to reduce the amount of alignment in that struct by 2.

```c++
struct Enemy
{
	uint32_t id;
	State state;
};
```

Meaning that this struct is even smaller.
# References
##### Main Notes
[[What is a memory block]]
[[Memory layout - The stack]]
[[Memory layout - The heap]]
#### Source Notes
[[Intro to Data Oriented Design for Games]]