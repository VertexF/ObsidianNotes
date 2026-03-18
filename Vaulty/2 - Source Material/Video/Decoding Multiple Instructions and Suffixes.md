# Reference https://www.computerenhance.com/p/decoding-multiple-instructions-and

First thing you'll need is the 8086 family manual which can be found [free online](https://ia600500.us.archive.org/23/items/bitsavers_intel80869lyUsersManualOct79_62967963/9800722-03_The_8086_Family_Users_Manual_Oct79.pdf) This is a continuation [[Instruction Decoding on the 8086]] please refer to that if you're confused.

As you may already know that when the **MOD** filed of the `mov` instruction isn't always `11` you are either doing a load or a store. Meaning you are reading more than just 2 bytes in, so the instruction grows in bytes depending on some of the fields.

If the **MOD** is `01` then you have 1 extra byte, if the **MOD** is `10` you have two extra bytes to deal with. This means that the instruction are heavily dependent on the previous bytes as you move along the instruction. The CPU has to do a lot of work to decode the instructions byte by byte. At the time this allowed for the instructions to be packed at the cost of complexity in the decode process.  

A load is, remember the first thing is the destination.
```c
mov ax, [75] //Special case
mov bx, [75] //Normal case
```

Warning, this is actually not as easy as it's made out `mov ax, [75]` is special case so the binary gonna be different.

The `[75]` is a pointer to a piece of memory. If we are using a 16bit register like `bx` we will automatically read out the `[75]` and `[76]` as well.

This is a store since the address look up comes first.
```c
mov [75], bx
```

Warning again, `mov [75], bx` is another special case which requires more binary to be parsed to do.

A more typical instruction looks like this
```c
mov bx, [bp+75]
```

Were we take a base pointer, in this case `bp` and do some addition on it. This is known as an **effective address calculation**. These are an intel speciality even though they have changed they are still there now. This is not general, you have to be careful, you can't do `[ax + bx + 27]` for example.
##### The byte encoded
When you have the move bytes like this `10010`**DW** the **W** is for wide 8/16-bit registers if the **D** is set to `1` then we are doing a **load** because the register goes first, if the **D** is `0` then we are doing a **store**.

In the second byte MOD REG R/M you need both the **MOD** and the **R/M** field to encode what the memory is.

The **R/M** field actually encodes the type of equation you have to do so is it an addition or what, that's in the **R/M**. The **MOD** field tells us if we have a displacement so the 27 in the `[bp+27]`.

So what does the MOD field tell us 

| MOD  | Encoding                       | Example    |
| ---- | ------------------------------ | ---------- |
| `00` | Do not have a displacement^    | `[bp]`     |
| `01` | Has a 8-bit displacement       | `[bp+8]`   |
| `10` | Has a 16-bit displacement      | `[bp+300]` |
| `11` | Is a register to register move |            |
The **MOD** field also tells if we have a disp-low or a disp-high connected to our bytes. So if the we have an 8-bit displacement we have a just disp-low if we have a 16-bit displacement we have both disp-low and disp-high.

^There is 1 case in **MOD** case `00` **DOES** have a displacement. If the **R/M** field is set to `110` you have something called a **direct address** and you have a 16-bit this displacement. This happens when you are just looking up the actual address value without an equation

```c
mov bx, [128]
```

I believe this value is in the displacement-low and displacement-high.

The **R/M** field encodes the 8 equation for you to use. We don't need to know the displacement but what are the register address that are going to used in this operation. 

|       | Register Encoding   | Register Encoding | Register Encoding |
| ----- | ------------------- | ----------------- | ----------------- |
| R/M   | MOD =`00`           | MOD =`01`         | MOD = `10`        |
| `000` | `bx+si`             | `bx+si+D8`        | `bx+si+D16`       |
| `001` | `bx+di`             | `bx+di+D8`        | `bx+di+D16`       |
| `010` | `bp+si`             | `bp+si+D8`        | `bp+si+D16`       |
| `011` | `bp+di`             | `bp+di+D8`        | `bp+di+D16`       |
| `100` | `si`                | `si+D8`           | `si+D16`          |
| `101` | `di`                | `di+D8`           | `di+D16`          |
| `110` | **DIRECT ADDRESS**^ | `bp+D8`           | `bp+D16`          |
| `111` | `bx`                | `bx+D8`           | `bx+D16`          |
These are the equation are allowed to use in your instruction. It's important to note that **bp** (base pointer), **sp** (stack pointer), **si** (source index), **di** (destination index), have no 8-bits parts, they are 16-bits or bust.

If you try to use `bp` with no effective address calculation, the **MOD** field is **NOT** `00`
```c
mov dx, [bp]
```

The ASM compiles this the **MOD** field to be `01` because `bp` for a `00` **MOD** field is resevered for direct address look ups such as `mov dx, [75]`

##### Immediate to register move
With the mov instruction encoding started with `1011`**W** **REG** this is a immediate to register move. This is used when we have a value that's directly encoded into the register.

This looks like this in ASM
```c
mov bx, 12
```

This is much more efficient than encoded a pointer to something in memory, since we don't have to do that look up. It's called an **immediate** because the value got fetched as part of an instruction. 

If **W** is set to `0` then we only have a disp-low to work with, if the **W** is set to `1` then the we have a disp-high too. Even if the immediate move can be encoded as 8-bit say with the number 12, if the register is 16-bits it will use both low and high, making high all zeros.
##### Immediate to register with an effective address calculation  
This instruction is encoded as `1100011`**W**

You have a problem with the move operation when you try to do something like this.
```c
mov [bp+75], 12 //Invalid
```

When we are doing an immediate to register move with a load operation the assembler doesn't know if we want the 8-bits or 16-bits. With things like `ax` and `ah` the letter tells us if it's 16-bits or 8-bits, but `[bp+75]` is this 8-bit or 16-bits? We can't know. This is because `[bp+75]` is an address calculation, that tells us were we are writing too and not how big it is. 

How this resolves when writing a ASM is to write either **byte** for 8-bits before the value or **word** before the value for 16-bits.

```c
mov [bp+75], byte 12 //8-bit mov
mov [bp+75], word 12 //16-bit mov
```

You can think of this as a cast. Then the assembler sets the **W** flag.

##### Signed displacement
When the assembler parses the instructions it encodes everything as unsigned, meaning if you say do 
```c
mov cx, -12
```

The CPU doesn't care unless it's a some kind of operation like addition. Meaning when it seeing `-12` it just see binary in a register it doesn't care. Meaning it's totally valid to print `-12` as the wrap around value as `65524` it's the literally the same in binary for a mov.

You can also do this for **effective address calculation** 
```c
mov ax, [bx + di - 37]
```

This is a signed displacement extension which is talked about in the manual.
###### Accumulator to memory
These are special **direct address** type instructions that have their own encoded **ONLY** if you're using the `ax` register.

The **load** were `ax` is first is when you have the encoded `1010000`**W** and 
The **store** were `ax` is second is when you have the encoded `1010001`**W** 

These operations do exactly the same thing but are encoded differently. 