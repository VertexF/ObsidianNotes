2026-03-17 16:30
Status: #baby 
Tags: [[assembly]]
# Effective address calculation

As you may already know that when the **MOD** filed of the `mov` instruction isn't always `11` you are either doing a load or a store. Meaning you are reading more than just 2 bytes in, so the instruction grows in bytes depending on some of the fields.

If the **MOD** is `01` then you have 1 extra byte, if the **MOD** is `10` you have two extra bytes to deal with, for the `100010`[[Register to register move operation|move instruction]]. This means that the instruction are heavily dependent on the previous bytes as you move along the instruction. The CPU has to do a lot of work to decode the instructions byte by byte. At the time this allowed for the instructions to be packed at the cost of complexity in the decode process.  

A **load** is, remember the first thing is the destination.
```c
//These are both special cases
mov ax, [75]
mov bx, [75]
```

Warning, this is actually not as easy as it's made out `mov ax, [75]` and `bx, [75]` are specials cases so the binary gonna be different.

The `[75]` is a pointer to a piece of memory. If we are using a 16bit register like `bx` we will automatically read out the `[75]` and `[76]` as well.

This is a **store** since the address look up comes first.
```c
mov [75], bx
```

Warning again, `mov [75], bx` is another special case which requires more binary to be parsed to do.

A more typical instruction looks like this
```c
mov bx, [bp+75]
```

Were we take a base pointer, in this case `bp` and do some addition on it. This is known as an **effective address calculation**. These are an intel speciality even though they have changed they are still there now. This is not general, you have to be careful, you can't do `[ax + bx + 27]` for example.
# References
##### Main Notes
[[Register to register move operation]]
[[Signed displacement for move]]
#### Source Notes
[[Decoding Multiple Instructions and Suffixes]]