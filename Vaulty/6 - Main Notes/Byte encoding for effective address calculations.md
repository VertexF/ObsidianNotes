2026-03-17 16:37
Status: #baby 
Tags: [[assembly]]
# Byte encoding for effective address calculations

When you have the move bytes like this `100010`**DW** the **W** is for wide 8/16-bit registers if the **D** is set to `1` then we are doing a **load** because the register goes first, if the **D** is `0` then we are doing a **store**.

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

This value is in the displacement-low and displacement-high of the instruction. This still has the same encoding but just breaks the displacement rules.

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
# References
##### Main Notes
[[Effective address calculation]]
#### Source Notes
[[Decoding Multiple Instructions and Suffixes]]