2026-09-06 08:00
Status: #baby
Tage: [[OS]] [[OS memory]]
# Introduction to paging

Paging is the idea of dividing memory up into fix size chunks to segement memory. This is a per process things, it allows us to split up memory that's more flexible and has no internal or external fragmentation. 

You have something called a **page table** that exists per-process that stores **page table entries**. The page table can be found within the CPU's MMU **base table register**. The page table This is used with the processes **virtual addresses** to translate a virtual address to physical address. 

A process has **virtual memory** which allows us to access addressable memory. The **address space** is (the physical memory that can be translated to by an **virtual address**). So you can have the an **address space** of 64-bytes and each **virtual page** of 16-byte giving you 4 virtual pages.

A **virtual address** is anything that exists to look up within in memory for a given process. This can be something like [[Effective address calculation]] or an instruction for a given piece of assembly.

Lets break down what the different parts of a **virtual address** when we are using paging to virtualise memory. With this instruction below we have a virtual address of `21` we will ingnore the instruction fetch that happend before it.

```c
movl [21] eax 
```

In a toy **address space** of 64-bytes in size with 4 virtual page table  

We view **physical memory** as a something that is split up into **physical frame**/**page frame** each **physical frame** A page table entry contains a **Physical Frame Number** (PFN). When we want to look up something in physical memory we first get the **Virtual Page Number** (VPN) which is found at the left most part of the **virtual address** for a given process. Using our **page table** with the (VPN) to get a **Physical Table Entry** (PTE) which contains a **Physical Frame Number** (PFN) which allows us to find the correct **physical frame** were the memory is to get the current byte we add the **offset** to the PFN. You then take the **Physical Frame Number** (PFN) multiply it by the size of a **Physical Frame Size**/**Virtual Page Size** (these are the same size) and then add the **offset** from the **virtual address** to that value. 

Here are the steps need to go through this.
Important to note physical frame and page size are the same size.

We take the virtual address and get and split it up into 2 chunks.
The virtual page number and the offset.
To work out offset we need to know how many bits we need for each physical frame. If we have a page size of 1KB then we have $2^{10}$ bits needed to represent that value.

Meaning our virtual address needs to have an offset of 10 bits to cross the whole physical frame top to bottom. Next we need to work out the physical frame number from the virtual address. Which the bits remain that are right to offset.

VPN      offset
(?? bits)(10 bits) 

You can work out how many bits you need to represent all the page table entries/physical frames by knowing how many physical frames/page table entries you have.

If you don't know that, then take the addressable space / page table size in bytes. For example if the page table size is 1KB and the addressable size is 16KB then you just do the division 

$$\frac{16KB}{1KB} = \frac{2^{10} \times16}{2^{10}} = \frac{2^{10} \times 2^{4}}{2^{10}} = \frac{2^{14}}{2^{10}} = 2^{4}$$

Then you take the $2^n$ and that $n$ tells you how many bits we need after the offset we need to store to do the address translation.

VPN      offset
(4 bits)(10 bits)

-----
To do the address translation from 1 virtual address to a physical address, you take the virtual page number get the physical frame number that's stored in the virtual page entry in the page table.

For example with this table and the virtual address of 0x00003385
Page Table (from entry 0 down to the max size)
```
  (       0)  0x80000018
  (       1)  0x00000000
  (       2)  0x00000000
  (       3)  0x8000000c
  (       4)  0x80000009
  (       5)  0x00000000
  (       6)  0x8000001d
  (       7)  0x80000013
  (       8)  0x00000000
  (       9)  0x8000001f
  (      10)  0x8000001c
  (      11)  0x00000000
  (      12)  0x8000000f
  (      13)  0x00000000
  (      14)  0x00000000
  (      15)  0x80000008
```
Virtual Address Trace
  VA 0x00003385 (decimal:    13189)

We take 0x00003385 covert it to binary `0000_0000_0000_0000_0011_0011_1000_0101` take the 10 bits from the right because we have $2^{10}$ sized physical frames/page sizes = `11_1000_0101 = 901` in decimal and the virtual page number which is four bits to the right of the offset `1100 = 12` in decimal. 

Then we look up the page table entry in page table at entry 12 which is `0x8000000f` if it's not all `0x000000000` then it's valid. The `0x8` at the beginning are just status bits we can ignore for now.

We take `0xf` which is = 15 that tells us which the physical frame we need to look up. We know how big the physical frame is so to get to the beginning of that physical frame position in RAM we multiply the the $15 * 1024$ (PFN * Physical Frame Size) which will get you to location 15360

Finally you take the offset from the virtual address and add it to the starting position of the physical frames value so $15360 + 901 = 16261$ or $(15 * 1024) + 901 = 16261$ which is the location in memory were 0x00003385 truly lives.
# References
##### Main Notes
[[Multi-level paging]]
#### Source Notes
