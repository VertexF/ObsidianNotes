2026-03-17 16:50
Status: #baby 
Tags: [[assembly]]
# Immediate to register move with an effective address calculation

This instruction is encoded as `1100011`**W**

You have a problem with the move operation when you try to do something like this.
```c
mov [bp+75], 12 //Invalid
```

When we are doing an immediate to register move with a load operation the assembler doesn't know if we want the 8-bits or 16-bits. With things like `ax` and `ah` the letter tells us if it's 16-bits or 8-bits, but `[bp+75]` is this 8-bit or 16-bits? We can't know. This is because `[bp+75]` is an address calculation, that tells us were we are writing too and not how big it is. 

How this resolves when writing a ASM is to write either **byte** for 8-bits before the value or **word** before the value for 16-bits.

```c
mov [bp+75], byte 12 //8-bit mov
mov [bp+75], word 12 //16-bit mov
```

You can think of this as a cast. Then the assembler sets the **W** flag.
# References
##### Main Notes
[[Immediate to register move]]
[[Signed displacement for move]]
#### Source Notes
[[Decoding Multiple Instructions and Suffixes]]