2026-03-17 16:52
Status: #baby 
Tags: [[assembly]]
# Signed displacement for move

When the assembler parses the instructions it encodes everything as unsigned, meaning if you say do 
```c
mov cx, -12
```

The CPU doesn't care unless it's a some kind of operation like addition. Meaning when it seeing `-12` it just see binary in a register it doesn't care. Meaning it's totally valid to print `-12` as the wrap around value as `65524` it's the literally the same in binary for a mov.

You can also do this for **effective address calculation** 
```c
mov ax, [bx + di - 37]
```

This is a signed displacement extension which is talked about in the manual. You just make the number 16-bits and treat it as a unsigned number, meaning that

```c
mov ax, [bx + di - 37]
mov ax, [bx + di + 65499]
```

Are the same operation due to wrapping numbers.
# References
##### Main Notes
[[Effective address calculation]]
[[Immediate to register move with an effective address calculation]]
[[Immediate to register move]]
#### Source Notes
[[Decoding Multiple Instructions and Suffixes]]
