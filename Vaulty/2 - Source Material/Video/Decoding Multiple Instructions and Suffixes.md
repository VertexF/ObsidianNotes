# Reference https://www.computerenhance.com/p/decoding-multiple-instructions-and

First thing you'll need is the 8086 family manual which can be found [free online](https://ia600500.us.archive.org/23/items/bitsavers_intel80869lyUsersManualOct79_62967963/9800722-03_The_8086_Family_Users_Manual_Oct79.pdf) This is a continuation [[Instruction Decoding on the 8086]] please refer to that if you're confused.

As you may already know that when the **MOD** filed of the `mov` instruction isn't `11` you are either doing a load or a store. Meaning you are reading more than just 2 bytes in, so the instruction grows in bytes depending on some of the fields.

If the **MOD** is `01` then you have 1 extra byte, if the **MOD** is `10` you have two extra bytes to deal with. This means that the instruction is are heavily on the bytes as you move along the instruction. The CPU has to do a lot of work to decode the instructions byte by byte. At the time this allowed for the instructions to be packed at the cost of complexity in the decode process.  