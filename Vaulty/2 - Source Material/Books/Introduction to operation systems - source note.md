# References Operating Systems in three easy steps

The operating system needs to virtual resource to make it easier for programs to run. This includes literally every piece of hardware, which is why sometimes an OS is called a virtual machine or a resource manager.

The OS provide system calls to make running programs possible.
##### Virtualizing The CPU
###### CPU
It's possible to run many programs at once that print 'A' every single and they all finish at exactly the same time. The OS is in charge of making the CPU virtualisation meaning that it's like we have an "infinite" number of CPUs on our PC. 

What if we want to run 2 programs at the exact same time? Well the OS **policy** handles that.
###### Memory
Memory of the data and instruction are held in memory. Each instruction fetch also gets the data. It's possible or a programs to allocate memory at exactly same memory. This because of OS virtualisation. Each program has it's own private virtual address space. The OS maps the virtual to the physical address and the physical address is a shared resource.
###### Persistence
When writing to disk the OS will do something called **journaling** or **copy-on-write** to be able to recover to a reasonable state after crashing. The OS has to keep track of simple and complex data structures to write to disk. It's also wait until possible to write things in bulk.
###### History
After an OS just being a group of libraries they progress to having system calls the part of the hardware that pick them up is called a **trap handler**, through an instruction called a **trap**. The trap handler raises the privilege level to kernel mode from user mode allowing things like file creation or whatever.