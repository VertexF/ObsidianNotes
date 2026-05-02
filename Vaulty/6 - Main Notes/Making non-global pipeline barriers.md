2026-04-27 13:57
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# Making non-global pipeline barriers

When using synchronisation2 you can use `VkDependencyInfo` which restrict what the memory barrier/execution barrier operates under.

```c
typedef struct VkDependencyInfo {
    VkStructureType                  sType;
    const void*                      pNext;
    VkDependencyFlags                dependencyFlags;
    uint32_t                         memoryBarrierCount;
    const VkMemoryBarrier2*          pMemoryBarriers;
    uint32_t                         bufferMemoryBarrierCount;
    const VkBufferMemoryBarrier2*    pBufferMemoryBarriers;
    uint32_t                         imageMemoryBarrierCount;
    const VkImageMemoryBarrier2*     pImageMemoryBarriers;
} VkDependencyInfo;
```

Here you can see 3 different types of barriers you can set up. Image and buffer are just used to restrict between buffer based action commands and image based action commands. The **VkMemoryBarrier2** is used for global barriers.

This will increase performance as your setting up hard synchronisation barriers for every time of memory that doesn't require it.

It's important to note that render passes are just a very round about way for image memory barriers. As when you eventually finish working with render passes they output a VkImageView inside a VkFramebuffer that goes to the VkSwapchain.
# References
##### Main Notes
[[What are the access masks]]
[[How to make GPU memory visible and available]]
[[The difference between source and destination stage mask]]
[[In-queue execution pipeline barriers]]
[[The pipeline stages]]
[[Vulkan command categories]]
[[Implicit synchronisation guarantees]]
[[Basic queue synchronisation]]
[[Vulkan command overlap]]
#### Source Notes
[[Vulkan synchronisation explained]]