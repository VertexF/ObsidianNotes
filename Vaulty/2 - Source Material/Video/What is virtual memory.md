# Reference https://www.youtube.com/watch?v=A9WLYbE0p-I

##### The three problems in memory
You have 3 major problems that virtual memory tries to solve with a layer of indirection. 
1) Security - It's possible that if every program used physical memory it would overwrite other program using that same piece of memory.
2) Running out of memory - If you loaded programs into RAM and you exceeded the address space and look for a piece of memory outside of the physical addresses you would get a uncoverable crash in the program.
3) Memory fragmentation - Again if you load programs into memory, it's possible that lots of little programs fill up the space in ways that quickly fill up the RAM address space.

So we need to give each program it's own virtual memory address space. Now we need to map the virtual address space to physical address space on the RAM. There is more memory than just RAM, there are other devices, so we call this physical address space. 

How do we solve the running out of memory? Well, because we aren't using the RAM directly, we the mapping can look at the disk instead. If we have ran out memory we do this 
1) Find that the mapping to physical memory doesn't exist.
2) The mapping tells us that this piece of memory is on the disk.
3) The we follow these steps
	1) The OS finds the oldest piece of physical memory that can fit what we need.
	2) Move that old piece of memory to disk.
	3) Now with the freed space we move what we needed out from the disk into RAM.
4) Now we update the mapping.

This is called swap memory. When we run out memory in RAM this is called a page fault. This of course is extremely slow.

How do we solve memory fragmentation? This is simple because each program has virtual memory it can be mapped into physical address space at any point even if it's fragmented.

How do you solve the security problem? This is simple again, if both programs try to write to the same memory address space the mapping maps them both to different physical address spaces. How do programs share memory then if they are totally isolated? Well in this case the virtual address space just maps to the physical address space were that data like a shared library is stored.
##### Page tables
The mapping is called page tables. So if you wanted to say load the address 64 from main memory they page table will map from that to the physical address space and give you whatever was mapped at that physical address space.

The CPU works with registers which are sized depending on the 32/64-bitness of the CPU. So on a 64-bit CPU the registers will be 64-bit wide. As the CPU works on words 

The problem is if we do to 32-bit for a moment if each page entry in the page table mapped from 1 address to 1 word the page table would be 4GBs which on a 32-bit process is the max amount of RAM that can be used anyway.

We need to make things into chunks

![[pagetablechunks.png]]

As you can see we take a virtual memory and divide it up into chunks of 4KBs. That is then set into our page table as it mapped from page 0 to page 3. Which is a 4KBs of memory. Now our page table is 4MBs per program! There is a trade off though, instead of loading 1 word we have to load 4KBs at a time. This is okay because you often have a lot of the same values that at in the same location.