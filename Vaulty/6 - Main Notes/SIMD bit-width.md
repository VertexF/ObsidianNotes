2026-03-07 07:33
Status: #baby 
Tags: [[performance]]
# SIMD bit-width

You have family of SIMD, SSE which is supported every were, AVX which is basically supported on everything and AVX-512 which isn't well supported. With each knew family the instructions get wider.
1) SSE is 4 instructions wide
2) AVX is 8 instructions wide
3) AVX-512 is 16 instructions wide

This is only true if you're using a 32-bit line size. The amount of instructions you can do per operation changes depending on the bits you're using so if you go down to 8 bits for example for SSE you'll get 16 operations per instruction.

The reason you can do this is because these families are just set by a bit-width.
1) SSE being 128
2) AVX being 256
3) AVX-512 being 512

You can use the bit-width in anyway you like.
# References
##### Main Notes
[[Introduction to SIMD]]
[[How to use SIMD with a summation loop]]
#### Source Notes
[[Single Instruction, Multiple Data]]