2026-03-17 16:56
Status: #baby 
Tags: [[assembly]]
# Accumulator to memory move

These are special **direct address** type instructions that have their own encoded **ONLY** if you're using the `ax` register.

The **load** were `ax` is first is when you have the encoded `1010000`**W** and 
The **store** were `ax` is second is when you have the encoded `1010001`**W** 

```c
mov ax, [2555]
mov ax, [16]
mov [2554], ax
mov [15], ax
```

These operations do exactly the same thing but are encoded differently.
# References
##### Main Notes
[[Byte encoding for effective address calculations]]
#### Source Notes
[[Decoding Multiple Instructions and Suffixes]]
