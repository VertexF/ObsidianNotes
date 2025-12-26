2025-12-23 19:32
Status: #baby 
Tags: [[OS]] [[OS memory]]
# Three problems with physical memory

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

How do you solve the security problem? This is simple again, if both programs try to write to the same memory address space the mapping maps them both to different physical address spaces. How do programs share memory then if they are totally isolated? Well in this case the virtual address space just maps to the physical address space were that data is. For example, where a shared library is stored.
# References
##### Main Notes
#### Source Notes
[[What is virtual memory]]