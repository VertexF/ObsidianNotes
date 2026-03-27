2026-03-17 16:16
Status: #baby 
Tags: [[assembly]]
# Instruction decode with mov

Instruction decode is a piece of work the CPU has to do to turn instructions to a mechanical idea that the CPU can operate and do. 

A mnemonic is a human friendly way of representing what the CPU can do. So like **mov** for example is a mnemonic of a copy instruction that the CPU can do. Move actually means copy. If we want to copy something from one register to another you would write it as

```c
mov ax, bx
```

How this works is that destination is **ax** and the **bx** is the source. When talking about assembly these are called **oprands**. So instruction oprand comma oprand, in order of destination, source. This is always the order for intel ASM.

It might help to think about this as a `move ax = bx` here we are using the = so to show that **bx** is the source.

How this will run on the CPU is like that
1) It run the code through an assembler
2) That encoded it to binary the x86 chip **instruction decoder** understands
3) The decoder then breaks down the binary down to tell the CPU what to do mechanically.
# References
##### Main Notes
[[Register to register move operation]]
[[Introduction to the 8086]]
#### Source Notes
[[Instruction Decoding on the 8086]]