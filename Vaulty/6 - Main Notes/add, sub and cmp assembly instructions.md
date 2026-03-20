2026-03-19 14:16
Status: #baby 
Tags: [[assembly]]
# add, sub and cmp assembly instructions

In the 8086 and all the way up to x64 the patterns of the throughout the whole instruction set are pretty similar. Meaning that when you want to do an operation, sometimes you don't even have to do a **mov** at all. You can actually bake your memory access into whatever memory access you are doing.

In non-8086 instruction to do an addition we might write things like this.
```c
mov ax, [bp]
mov bx, [bp + 2]
add ax, bx
```

However we might do something different. Lets say we just wanted to do the addition if we want to use **bx** for something else or we wanted to keep **bx** with another piece of data that's already in **bx**.

Nice the `add` instruction is encoded pretty much exactly the same as a `mov` instruction you can encode anything you want in the source place.

Meaning that we can do this.
```c
mov ax, [bp]
add ax, [bp + 2]
```

This skips the second move instruction we added. This is only possible because the encoded for `add` looks exactly the same as for `mov`. The only difference is the opcode at the beginning of the instruction. This is very important to notice. This extends beyond `add` but `sub`, `cmp` all these have the same idea. 

This all means that when you're writing a decoder for `add`, `cmp` and `sub` you can re-use the same ideas across the same instructions. 

Some with `add`, `cmp` and `sub` the the immediate instruction like with the [[Immediate to register move with an effective address calculation]] which has a second byte containing `000` in the middle. These instructions have the same opcode for each of them `100000`**SW** but the way you tell these opcode apart is with the second byte, that encodes the pattern of 
1) `000` for `add`
2) `101` for `sub`
3) `111` for `cmp`

If you look at the [[Register to register move operation]] and [[Effective address calculation]] take that knowledge and notice for the same instruction for `add`, `cmp` and `sub` also encode the same 3 bytes listed but now in the opcode. 
# References
##### Main Notes
[[Immediate to register move with an effective address calculation]]
[[Register to register move operation]]
[[Effective address calculation]]
[[Conditional jumps]]
[[Signed extension]]
#### Source Notes
[[Opcode Patterns in 8086 Arithmetic]]