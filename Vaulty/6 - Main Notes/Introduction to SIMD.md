2026-03-07 07:30
Status: #baby 
Tags: [[performance]]
# Introduction to SIMD

Single instruction, multiple data is all about trying to reduce the amount of instructions you are trying to get the CPU to do all while doing the same amount of work.

The idea being is that if we have this ASM
```c
ADD A, input[0]
ADD B, input[1]
ADD C, input[2]
ADD D, input[3]
```

CPU designer have had this idea because typically most people do a tonne of the same operations. Maybe there is a way to do 1 operation for all these **ADD's** in one go with the data and writing into registers at the same time.
```c
SUPER_ADD A,B,C,D input[0], input[1], input[0], input[1]
```

So x64 ASM you have the SSE, or Streaming SIMD Extensions which came out first to do this. So the most basic SSE instruction is **PADDD** said P-ADD-D The extra **D** is because it's a D word meaning it's a 32-bit adding operation and **P** means packed, because we are packing things together. 

This takes 4 slots instead of 1 to add things together. So we do 4 loads and add them into 4 accumulators. Meaning when we do a PADDD it's like we do 4 ADD's This is were the idea of lanes being aligned with registers. 

You can think of these instructions like vector operations, which might be useful for when you're trying to use them. For example with PADDD it would be like this.

| A | + | input[0] |    | A + input[0] |
| A | + | input[1] |    | A + input[1] |
| A | + | input[2] | = | A + input[2] |
| A | + | input[3] |    | A + input[3] |

Even when you are [[Solving the serial dependency problem]] the CPU still has to do a bunch of work to work to check if things are dependent or not and then send that to the right place on the CPU. When you use SIMD you can save a bunch of work because you do that work for every 4 adds rather than for every **ADD**.
# References
##### Main Notes
[[SIMD bit-width]]
[[Solving the serial dependency problem]]
#### Source Notes
[[Single Instruction, Multiple Data]]
