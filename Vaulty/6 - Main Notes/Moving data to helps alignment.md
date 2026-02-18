2026-02-11 12:02
Status: #baby 
Tags: [[performance]] [[DOD]]
# Moving data to helps alignment

This is very simple if you have a struct with an alignment of like 8 and you want to help your CPU when it's fetch memory from RAM or the cache increase the density of your structs by making sure that the size is as small as possible. Just put the smallest variables at the bottom.

```c++
//Bad - Alignment 8 Size 88
struct Entity
{
	bool isDead;
	uint64_t id;
	char name;
	uint32_t items[15];
	bool isCharging;
};

//Good - Alignment 8 Size 72
struct Entity2
{
	uint64_t id;
	uint32_t items[15];
	char name;
	bool isDead;
	bool isCharging;
};
```

This can be a little fiddly to get right sometimes and sometimes the size can go up or not change. Just make sure you test thing to see if what you did is actually helping.
# References
##### Main Notes
[[Aligning your structs for better performances]]
[[Reducing struct memory alignment]]
[[Storing bools out of bounds]]
#### Source Notes
[[Intro to Data Oriented Design for Games]]