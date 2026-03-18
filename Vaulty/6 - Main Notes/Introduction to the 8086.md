2026-03-17 16:12
Status: #baby 
Tags: [[assembly]] [[CPU]]
# Introduction to the 8086

The 8086 was the first x86 CPU. In the CPU there are registers. On the 8086/88 the register was a literal space on the die for storing info. Each register was 16-bits worth of storage. The flow back then was really simple, you moved things from main memory into registers, do a thing then move it back out again. This is still what it's like today but the registers aren't that literal.

In this course you're going to build simulate a 8086 CPU to a limited to degree, with the idea that if you know how to simulate a CPU you know how it works.

First thing you'll need is the 8086 family manual which can be found [free online](https://ia600500.us.archive.org/23/items/bitsavers_intel80869lyUsersManualOct79_62967963/9800722-03_The_8086_Family_Users_Manual_Oct79.pdf) just google it. In page 259 or chapter 4_21 in the PDF I have and starting with 257 you'll find the beginning of the 8086 instruction encoding tables.
# References
##### Main Notes
[[Instruction decode with mov]]
[[Register to register move operation]]
#### Source Notes
[[Instruction Decoding on the 8086]]