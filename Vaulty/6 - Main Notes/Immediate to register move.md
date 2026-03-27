2026-03-17 16:47
Status: #baby
Tags: [[assembly]]
# Immediate to register move

With the mov instruction encoding started with `1011`**W** **REG** this is a immediate to register move. This is used when we have a value that's directly encoded into the register.

This looks like this in ASM
```c
mov bx, 12
```

This is much more efficient than encoded a pointer to something in memory, since we don't have to do that look up. It's called an **immediate** because the value got fetched as part of an instruction. 

If **W** is set to `0` then we only have a disp-low to work with, if the **W** is set to `1` then the we have a disp-high too. Even if the immediate move can be encoded as 8-bit say with the number 12, if the register is 16-bits it will use both low and high, making disp-high all zeros.

What about signed numbers? Well [[Signed displacement for move]] has you covered. 
# References
##### Main Notes
[[Effective address calculation]]
[[Byte encoding for effective address calculations]]
[[Signed displacement for move]]
#### Source Notes
[[Decoding Multiple Instructions and Suffixes]]
