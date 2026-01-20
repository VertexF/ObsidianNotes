2025-12-25 07:39
Status: #baby 
Tags: [[OS]] [[OS memory]]
# Memory layout - The stack

![[memorylayout.png]]

This is a piece of dynamic memory that grows as more things are added to the stack. These are variables like local variables, function parameters, and return addresses. Each function add a stack frame when the function is ran, it then pops off the stack once the function has finished. This is done by a stack pointer that moves up as things are added and moves down as things are popped. Which is why it's called the stack it's FIFO.

When the stack and the heap meet the program is out of free memory.
# References
##### Main Notes
[[Memory layout - Text section]]
[[Memory layout - Initialised data]]
[[Memory layout - BSS uninitialised data]]
[[Memory layout - The heap]]
#### Source Notes
[[Memory Layout of a C Program]]