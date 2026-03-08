2026-03-06 15:32
Status: #baby 
Tags: [[performance]]
# Solving the serial dependency problem

If we list out the instructions for these additions we see that each addition is dependent on each other because it's a rolling sum for this loop

```c
int array[4096] = { /*random values in here*/ };
int sum = 0;
for(int i = 0; i < 4096; ++i)
{
	sum += array[i];
}
```

We get this

```c
ADD A, array[i]
ADD A, array[i + 1]
ADD A, array[i + 2]
ADD A, array[i + 3]
```

Each time we go and do this addition we have to fetch what **A** is BUT importantly we don't know what **A** is until we have finished the last **ADD**. Meaning that we have a big long serial dependency happening here. So the CPU has to wait for each instruction to finish before it moves on.

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
int result = sum1 + sum2 + sum3 + sum4;
```


Now the CPU can add A, B, C and D in parallel to each other because they are not dependent on each other. This gets us up to 1.7 and 1.94 with both loops by breaking up the dependency chains.
# References
##### Main Notes
[[The cost of loop over head]]
[[Solving for serial dependencies with SIMD]]
[[After thoughts on IPC for a rolling sum]]
#### Source Notes
[[Instructions Per Clock]]