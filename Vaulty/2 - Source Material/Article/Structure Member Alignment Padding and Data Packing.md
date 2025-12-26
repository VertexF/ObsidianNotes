# Reference https://www.geeksforgeeks.org/c/structure-member-alignment-padding-and-data-packing/

Each data type has a natural alignment requirement that's dicated by the CPU architecture. Historically byte address memory sequentially.
##### What is a memory block
A memory block is a logical unit of memory that is hardware dependent. The memory is determined by a memory controller and a physical organisation of the hardware memory slots. The memory controller is on the CPU and controlls what the CPU needs to access. In hardware the muiltple columns and rows of storage units. The rows and columns we are talking about here are the actually memory you can find on individual RAM chips, basically the square chips on a stick of RAM.

Here is a picture of 8 groups of memory banks on a RAM memory die 

![[memorybanksgroups.png]]

The groups can be sub-divided into 4 memory branks

![[memorybanks.png]]

This in totally means that you have 32 memory banks 8 x 4 = 32.

Remember the size of these units is controlled my the layout of the chips and the memory controller so a storage unit is almost always spread across mulitple RAM chips.
##### Data Alignment in Memory
A byte is 8bit if you are using 1 byte to find addresses in memory. 