2026-03-19 14:19
Status: #baby 
Tags: [[assembly]]
# Conditional jumps

You have instructions like `jnz` jumps to somewhere in code. These prefex **j** to be jump and **le** for less than, this type of instruction would be called `jle`, it's best to look at the instruction manual what the condition is.

To decode this you just look at the opcode and the 8 bit displacement. It's very simple to decode unlike everything else. There are no-conditional jumps and they work different to `jnz`. 
# References
##### Main Notes
[[add, sub and cmp assembly instructions]]
#### Source Notes
[[Opcode Patterns in 8086 Arithmetic]]