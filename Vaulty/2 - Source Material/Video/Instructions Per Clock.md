# Reference https://www.computerenhance.com/p/instructions-per-clock

When it comes to slow downs, it's possible that you actually make the program even slower by doing because you are telling the CPU do things in a slow way even if you have no waste.

Here we are **IPC** instructions per-clock and **ILP** instruction level parallelism basically mean the same thing. **IPC** is the average instructions per cycle the CPU calculates, so is for all instructions that the CPU does. **ILP** is just a the general that a CPU can do a certain number of instructions per cycle.

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

Now we are adding things 4 at a time rather than 1 at a time. If you do this you'll get 0.99 adds per-cycle rather than 0.801 adds per cycle for both. However there is something else going on here.

What ends up happening with this is that the CPU tries to do things in parallel with the CPUs **ILP** instruction level parallelism ability. What the CPU is able to do is it looks a the comparison and the add operation at the same time because **ADD a, b** and **COMPARE i, 4096** have nothing to do with each other.
##### Serial dependency
If we list out the instructions for these additions we see that each addition is dependent on each other because it's a rolling sum

```c
ADD A, array[i]
ADD A, array[i + 1]
ADD A, array[i + 2]
ADD A, array[i + 3]
```

Each time we go and do this addition we have to fetch what **A** is BUT importantly we don't know what **A** is until we have finished. Meaning that we have a big long serial dependency happening here. So the CPU has to wait for each instruction to finish before it moves on.

So for adding numbers up like this you have ultimate flexibility when it comes to adding, we can add up all the odd and even number together and then sum those up at the end for example. This would mean we would have something that looks like this

```c
int array[4096] = { /*random values in here*/ };
int sum1 = 0;
int sum2 = 0;
int sum3 = 0;
int sum4 = 0;
for(int i = 0; i < 4096; i+=4)
{
	sum1 += array[i + 0];
	sum2 += array[i + 1];
	sum3 += array[i + 2];
	sum4 += array[i + 3];
} 
int result = sum1 + sum2 + sum3 + sum4;
```

The ASM would look something like this
```c
ADD A, array[i]
ADD B, array[i + 1]
ADD C, array[i + 2]
ADD D, array[i + 3]
COMPARE i, 4096
ADD i, 1
JMP
ADD A, array[i]
ADD B, array[i + 1]
ADD C, array[i + 2]
ADD D, array[i + 3]
```

SIDENOTE - This is technically faster. Here we are just doing pointer arithmetic instead of using a for loop.  
```c
int array[4096] = { /*random values in here*/ };
int sum1 = 0;
int sum2 = 0;
int sum3 = 0;
int sum4 = 0;
int count = 4096 / 4;
while(count--)
{
	sum1 += array[0];
	sum2 += array[1];
	sum3 += array[2];
	sum4 += array[3];
	array += 4;
}
```


Now the CPU can add A, B, C and D in parallel to each other because they are not dependent on each other. This gets us up to 1.7 and 1.94 with both loops by breaking up the dependency chains.

##### After thoughts
This is great, but this isn't exactly how the real world would work, for 2 reasons. 
1) We were very easily able to break up the dependency chain because of what we were doing. This doesn't mean you can break dependency chains, it will require more thought that's all.
2) We are having to do a tonne of loads per cycle which isn't normal. Typically we would do more computations per loop cycle than this and the CPUs are optimised for computations in loops not loads.
