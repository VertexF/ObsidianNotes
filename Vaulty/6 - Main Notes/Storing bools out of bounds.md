2026-02-11 12:00
Status: #baby 
Tags: [[performance]] [[DOD]]
# Storing bools out of bounds

Bools in structs are annoying because the struct have to be padded by their largest value and a bool is 1 bit of information so the density of the struct drops dramatically if you have a lot of bools in your data. 

The questions you want to be asking when you see a struct is
1) Is it possible that your code already knows if something is true or false before it even gets to that point in the code?
2) Do you really need to know at the last possible moment if something is true or false? 

One way of solving bool in struct issue is by pulling the bool out of bounds for example

```c++
struct Monster
{
	uint32_t health;
	bool isDead;
};
Array<Monster> monsters;
```

It's possible that we never need to check this variable because we always know based on context if a monster is dead or alive. We could just store a list of dead and alive monsters, meaning that the bool is stored out-of-bounds of the struct by the context of there being 2 arrays.

```c++
struct Monster
{
	uint32_t health;
};
Array<Monster> deadMonsters;
Array<Monster> aliveMonsters;
```
# References
##### Main Notes
[[Aligning your structs for better performances]]
[[Reducing struct memory alignment]]
[[Moving data to helps alignment]]
#### Source Notes
[[Intro to Data Oriented Design for Games]]