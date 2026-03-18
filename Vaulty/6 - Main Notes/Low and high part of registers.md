2026-03-17 16:26
Status: #baby 
Tags: [[assembly]]
# Low and high part of registers

Registers store 2 bytes in them. If we are encoding registers that take 16-bits we then copy the all the data. Each byte either higher or lower, if we copying the 8-bits we need to encode either the low or the high parts of the register. This is done with
1) `mov ax, bx` for the entire 16-bits to be copied.
2) `mov al, bl` for the lower 8-bit parts to be copied.
3) `mov ah, bh` for the higher 8-bit parts to be copied.
# References
##### Main Notes
[[Register to register move operation]]
#### Source Notes
[[Instruction Decoding on the 8086]]
