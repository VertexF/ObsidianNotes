2026-03-06 15:36
Status: #baby 
Tags: [[performance]]
# After thoughts on IPC for a rolling sum

When reading both [[The cost of loop over head]] which reduces the loops over head and [[Solving the serial dependency problem]] which breaks up a dependency chain, they are using this example

```c
int array[4096] = { /*random values in here*/ };
int sum = 0;
for(int i = 0; i < 4096; ++i)
{
	sum += array[i];
}
```

The speed ups are valid and will work but there are 2 things that needs to be said about the for loop example.

1) We were very easily able to break up the dependency chain because it's a rolling sum. This doesn't mean you can break dependency chains, it will require more thought to do.
2) We are having to do a tonne of loads per cycle which isn't normal. With **array[i]**. Typically we would do more computations per loop cycle than this and the CPUs are optimised for computations in loops not loads.
# References
##### Main Notes
[[The cost of loop over head]]
[[Solving the serial dependency problem]] 
#### Source Notes
[[Instructions Per Clock]]