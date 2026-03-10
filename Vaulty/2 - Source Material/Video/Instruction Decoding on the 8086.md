# Reference https://www.computerenhance.com/p/instruction-decoding-on-the-8086

The 8086 was the first x86 CPU. In the CPU there are registers. On the 8086/88 the register was a literal space on the die for storing info. Each register was 16-bits worth of storage. The flow back then was really simple, you moved things from main memory into registers, do a thing then move it back out again. This is still what it's like today but the registers aren't that literal.

In this course you're going to build simulate a 8086 CPU to a limited to degree, with the idea that if you know how to simulate a CPU you know how it works.
##### Instruction Decode
Instruction decode is a piece of work the CPU has to do to turn instructions to a mechanical idea that the CPU can operate and do. 

A mnemonic is a human friend way of representing what the CPU can do. So like **mov** for example is a mnemonic of a copy instruction that the CPU can do. Move actually means copy. If we want to copy something from one register to another you would write it as

```c
mov ax, bx
```

How this works is that destination is **ax** and the **bx** is the source. When talking about assembly these are called **oprands**. So instruction oprand comma oprand, in order of destination, source. This is always the order for intel ASM.

It might help to think about this as a `move ax = bx` here we are using the = so to show that **bx** is the source.

How this will run on the CPU is like that
1) It run the code through an assembler
2) That encoded it to binary the x86 chip **instruction decoder** understands
3) The decoder then breaks down the binary down to tell the CPU what to do mechanically.
##### The register to register move operation
The x86 **mov** operation is encoded in 2 bytes (16-bit). You instructions in 1 byte (8-bits) at a time from right to left. 

In the first byte there is a bit pattern that tells us that it's a **mov** instruction, which is `100010` followed by 2 bit flags D and W. That gives us our first byte.

The second byte is made up of 3 component parameters of the **mov**
1) **MOD** field, made up of 2 bits
2) **REG** field, made up of 3 bits
3) **R/M** field, made up of 3 bits.

Together these 2 bytes tells us what type of **mov** operation we are going to do.

The **MOD** tells us if this is memory operation or a register operation. Meaning that if we are trying to move memory from 1 register to another, move memory from a register to RAM or RAM to register this all gets encoded in the **MOD** field. To copy 1 value to another register the **MOD** field should be `11`.

When the **MOD** is `11` the **REG** and **R/M** field encode registers. The **REG** is the register field. **R/M** have two meaning, **R** meaning register, **M** meaning memory operation.

Since the destination and source need to be in order, we need to know were the registers go either **R/M** or **REG**. To work this out we need to go back to the first byte. After the **mov** encoded you have the **D** field which tells you the order.
1) If the **D** bit is `0` **REG** is NOT the destination.
2) If the **D** bit is `1` **REG** IS the destination

If we are `0` here that means the register goes second in the **mov** instruction. If it's `1` then it comes first.

The **W** bit tells us if this is a 16-bit copy or a 8-bit copy. **W** stands for wide. The **W** also tells us if the register we are copying to is 16-bit or 8. The registers are either 8 or 16-bit, so the tells us if we are using 8/16-bit.

Finally with **REG** and **R/M** are 3 bits which encode which register they are going to use. Include the **W** we know what register to move to what. 
##### High and low part of registers
Registers store 2 bytes in them. If we are encoding registers that take 16-bits we then copy the all the data. Each byte either higher or lower, if we copying the 8-bits we need to encode either the low or the high parts of the register. This is done with
1) `mov Ax, Bx` for the entire 16-bits to be copied.
2) `mov Al, Bl` for the lower 8-bit parts to be copied.
3) `mov Ah, Bh` for the higher 8-bit parts to be copied.
##### How to start simulating the 8086 process
First thing you'll need is the 8086 family manual which can be found [free online](https://ia600500.us.archive.org/23/items/bitsavers_intel80869lyUsersManualOct79_62967963/9800722-03_The_8086_Family_Users_Manual_Oct79.pdf) just google it. In page 259 or chapter 4_21 in the PDF I have and starting with 257 you'll find the beginning of the 8086 instruction encoding tables.

