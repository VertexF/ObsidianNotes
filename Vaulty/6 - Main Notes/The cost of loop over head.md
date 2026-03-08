2026-03-06 15:28
Status: #baby 
Tags: [[performance]]
# The cost of loop over head

```c
int array[4096] = { /*random values in here*/ };
int sum = 0;
for(int i = 0; i < 4096; ++i)
{
	sum += array[i];
}
```

Now if you look at this loop in detail you can see that it's doing more than just **+=** 
1) it's having to do a "load" with the code **array[i]**
2) It's having to compare something every loop with the **<** operator
3) It's having to do a second add with **++i**

So it's really doing 4 instructions per-loop in this C code including the **+=**

When you do this there is loop overhead. If you look at the **<** and the **++i** operations there is a cost to these guys. So what we could do is "unroll" loop like this.

```c
int array[4096] = { /*random values in here*/ };
int sum = 0;
for(int i = 0; i < 4096; i+=2)
{
	sum += array[i + 0];
	sum += array[i + 1];
}
```

Or this
```c
int array[4096] = { /*random values in here*/ };
int sum = 0;
for(int i = 0; i < 4096; i+=4)
{
	sum += array[i + 0];
	sum += array[i + 1];
	sum += array[i + 2];
	sum += array[i + 3];
}
```

Now we are adding things 4 at a time rather than 1 at a time. If you do this you'll get 0.99 adds per-cycle rather than 0.801 adds per cycle for both loops. However there is something else going on here.

What ends up happening with this is that the CPU tries to do things in parallel with the CPUs **ILP** instruction level parallelism ability which why it's getting stuck at 1 add per cycle. What the CPU is able to do is it looks a the comparison and the add operation at the same time because **ADD a, b** and **COMPARE i, 4096** have nothing to do with each other.
# References
##### Main Notes
[[What does IPC or ILP mean]]
[[Solving the serial dependency problem]]
#### Source Notes
[[Instructions Per Clock]]