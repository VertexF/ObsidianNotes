2025-12-03 11:29
Status: #baby 
Tags: [[OS]]
# Introduction to operation system

The operating system needs to virtual resource to make it possible for muiltple programs to run. The OS makes literally makes every piece of hardware a programming uses virtual, which is why sometimes an OS is called a virtual machine or a resource manager.

The OS provide system calls to make running programs possible. Like a "writing to disk" system call.
###### Virtualizing The CPU
It's possible to run many programs that all at once that print 'A' every second and they all finish at exactly the same time. The OS is in charge of making the CPU virtualisation meaning that it's like we have an "infinite" number of CPUs on our PC. 

What if we want to run 2 programs at the exact same time? Well the OS **policy** handles that.
###### Memory
Memory of the data and instruction are held in memory. Each instruction fetch also gets the data. It's possible or a programs to allocate memory at exactly same memory as other programs. This because of memory virtualisation. Each program has it's own private virtual address space. The OS maps the virtual to the physical address and the physical address is a shared resource.
###### Persistence
When writing to disk the OS will do something called **journaling** or **copy-on-write** to be able to recover to a reasonable state after crashing. The OS has to keep track of simple and complex data structures to write to disk. It's also wait until possible to write things in bulk.
###### History
An OS used to be a bunch of libraries you could use until it progressed to having system calls. The part of the hardware that picks them up is called a **trap handler**, through an instruction called a **trap**. The trap handler raises the privilege level to kernel mode from user mode allowing things like file creation or whatever. You also have a special **return-from-trap** instruction which reverts things back to user mode while simultaneously passing control back to the application were things left off.
# References
##### Main Notes
#### Source Notes
[[Introduction to operation systems - source note]]
