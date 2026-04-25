2026-03-07 13:35
Status: #baby 
Tags: [[performance]]
# What is the cache

When you are doing something like this 

```c
int array[4096] = { /*random values in here*/ };
void addNoSIMD(uint32_t count, uint32_t *input)
{
	int sum = 0;
	for(int i = 0; i < count; ++i)
	{
		sum += input[i];
	}
}
```

There are many other things that can be done to improve how many adds you can do per cycle. You have to consider load and stores when it comes to algorithm. A load is when you get something from memory and a store is when you put something in memory.

If for example the value **input[0]** in a **ADD A, input[0]** isn't in the register to do the addition, we have to wait for that to happen first. This is another dependency when we do that operation on the load. This is what caching is concerned with.

As you may already know going to main memory is really slow compared to the **ADD** circuitry. Meaning that when we did the work with [[Solving for serial dependencies with SIMD]] we would've got anywhere close to the speeds we would've got to if we had to go to main memory.

When you have something like this
```c
ADD A, input[0]
```

The **A** values isn't going anywhere in a rolling sum. So **A** in this case, belongs to something called the **register file**. The register file feeds data into the instructions extremely quickly, but it's very small only able to store about 100 values or so.

The **input[0]** needs to fetch and it will first look to the **register file**, then the L1 (level1) cache, then the L2 cache, L3 cache if you're CPU has it then, main memory, then your hard disk.

It's important to note that each CPU you have core and each level cache. So on a intel skylake CPU, you have L1 and L2 for each core and the L3 is shared between cores.

You'll have to look up the hardware your targeting for a given CPU to really know what the cache situation looks like. 
# References
##### Main Notes
[[How the cache effects performance]]
#### Source Notes
[[Caching]]