2026-03-17 16:26
Status: #baby 
Tags: [[assembly]]
# Low and high part of registers

Registers store 2 bytes in them. If we are encoding registers that take 16-bits we then copy the all the data. Each byte either higher or lower for some registers, if we copying the 8-bits we need to encode either the low or the high parts of the register. This is done with
1) `mov ax, bx` for the entire 16-bits to be copied.
2) `mov al, bl` for the lower 8-bit parts to be copied.
3) `mov ah, bh` for the higher 8-bit parts to be copied.

The registers **bp**, **sp**, **si** and **di** all don't have a low or high part to them. What ends up being using depends on the **W** field encoding. Which tells us what gets copied to what

| RED   | W=0  | W=1  |
| ----- | ---- | ---- |
| `000` | `al` | `ax` |
| `001` | `cl` | `cx` |
| `010` | `dl` | `dx` |
| `011` | `bl` | `bx` |
| `100` | `ah` | `sp` |
| `101` | `ch` | `bp` |
| `110` | `dh` | `si` |
| `111` | `bh` | `di` |

# References
##### Main Notes
[[Register to register move operation]]
#### Source Notes
[[Instruction Decoding on the 8086]]
