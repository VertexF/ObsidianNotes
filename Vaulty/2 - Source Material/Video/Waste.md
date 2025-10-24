# Reference www.computerenchance.com/p/waste

This is to do reducing the amount of instructions. This really makes things way slower. When you have waste you are running instructions on the CPU that have nothing to do with the task at hand.

To talk about waste you need to know what `ADD A, B` does. It simply takes A and B which can be registers or memory address and looks into them adds two ints together. Replacing A with the result.

You need to know `LEA C, [A+B]` LEA means load effect address and this also can add two numbers together. The reason for the name is that it once was only meant for doing address look up operations. What is does is that is adds A and B together and writes it to a different place in memory called C.
1) `ADD A, B` is equal to `A += B;`
2) `LEA C, [A+B] ` is equal to `C = A + B;`

When you write an add operation in language you have to do something like this on the CPU. 

If you break a break point in VS. You can left click and in that menu click on `Go To Disassembly` and VS will bring up a window for you to look at the assembly and the code in the same window.

9:02