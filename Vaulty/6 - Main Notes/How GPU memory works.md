2026-04-29 07:40
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# How GPU memory works

L2 memory on the GPU is considered **available** to but to be used by the warps, it needs to be moved to L1 memory to be considered **visible**. The L2 cache is connected to the GPU DDR memory which is connected to the CPU memory controller in some way through PCI-e or UMA

This vulkan memory model of **visible** and **available** is just the mental model that surrounds incoherent caches which is different from CPU and RAM which is always coherent on x86 CPUs.

Lets say that in warp work has done has written some colour values out after being processed. This would be it's **visible** memory but not **available** memory, meaning it can't be used by any warp, unless the memory is pushed to L2. If memory is **available** it needs to be pulled to L1 cache becoming **visible**.

This is why we both use a source and destination access to mask to make sure the memory transfer is completed before we move on with further execution.

When we are making memory available we are flushing the caches, when we are making memory visible we are invalidating caches. (This is just the more general term used.)
# References
##### Main Notes
[[What are the access masks]]
[[How to make GPU memory visible and available]]
[[How memory layout transitions work]]
#### Source Notes
[[Vulkan synchronisation explained]]