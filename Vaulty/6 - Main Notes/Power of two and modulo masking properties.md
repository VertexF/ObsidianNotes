2026-09-08 15:26
Status: #baby 
Tags: [[computer science]]
# Power of two and modulo masking properties

If you have a power of 2 number which can be written in binary as a single digit 1 at some place with no other 1 (apart from zero) `0010` or 
`0010 0000 0000` for example.

If you do a modulo operation with one of these powers of 2 numbers you actually end up masking off all the binary digits include were the 1 lines and everything to the right.

For example lets take the number `0x1bc9` and `0x400` and modulo these numbers together, you get this
```
0x1bc9 = 0001_1011_1100_1001 %
0x0400 = 0000_0100_0000_0000 
=
0x03c9 = 0000_0011_1100_1001
```

Since `0x0400` is equal to 1024 or `0100 0000 0000` in binary removes all the values to the right on the number `0x1bc9` as if we masked off the last to `1` values.

This works for larger numbers too. It doesn't matter how many 1 digits are to the right of the value.
```
0xffbc9 = 1111_1111_0110_1101_1010 %
0x0400  =      0000_0100_0000_0000 
=
0x02da  = 0000_0000_0010_1101_1010
```
# References
##### Main Notes
#### Source Notes
