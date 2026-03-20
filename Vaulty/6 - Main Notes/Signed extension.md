2026-03-19 16:59
Status: #baby 
Tags: [[assembly]]
# Signed extension

When you're doing an instruction with an immediate, so something without a move like.
```c
add ax, word -7
```

It's possible that when you're using **word** but you're using a signed byte amount of data but you want to encode it as 16bit.

Then you need to so a sign extension to the data to turn our 8bit -7 to a 16bit -7. A negative sign extension is when you add all 1's in front of the data instead of 0's. For example -7 is in 8bits is
`111111001` where the first bit is the sign. If you sign extend this out to 16bits you need to add in all 1's

So `1111111110000111` is negative 7, because how it's worked out is that you start with a negative -65534 then you add all the other numbers that are in the binary. For example, lets take a signed 4 bit number and `1110` that would be in decimal, 
-8 + 4 + 2 + 0 = -2

This all means when you want to extend a binary number from 8 to 16 or more you need to know if it's signed. Then you extend it by adding 1's at the most significant bit for signed or 0's if it's unsigned or just a positive number.
# References
##### Main Notes
[[add, sub and cmp assembly instructions]]
#### Source Notes
[[Casey Q&A 5]]