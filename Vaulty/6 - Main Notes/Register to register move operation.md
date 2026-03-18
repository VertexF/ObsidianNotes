2026-03-17 16:19
Status: #baby 
Tags: [[assembly]]
# Register to register move operation

The x86 **mov** operation is encoded in 2 bytes (16-bit). The instruction is read in 1 byte (8-bits) at a time from right to left. 

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

The **W** bit tells us if this is a 16-bit copy or a 8-bit copy. **W** stands for wide. The **W** tells us if the register we are copying to is 16-bit or 8. The [[Low and high part of registers|registers are either 8 or 16-bit]], so the tells us if we are using 8/16-bit. 

Finally with **REG** and **R/M** are 3 bits which encode which register they are going to use. Include the **W** we know what register to move to what. 
# References
##### Main Notes
[[Low and high part of registers]]
[[Instruction decode with mov]]
#### Source Notes
[[Instruction Decoding on the 8086]]