2026-04-29 12:09
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# How memory layout transitions work

Just to start according to this [blog](https://themaister.net/blog/2019/08/14/yet-another-blog-explaining-vulkan-synchronization/) VkImage**Buffer**Barrier aren't interesting and no GPU really cares about them.

When you're working with `VkImageMemoryBarrier` you have to change the image layout at some point. 

```c++
typedef struct VkImageMemoryBarrier {
    VkStructureType sType;
    const void* pNext;
    VkAccessFlags srcAccessMask;
    VkAccessFlags dstAccessMask;
    VkImageLayout oldLayout; //Layout change here
    VkImageLayout newLayout; //and here
    uint32_t srcQueueFamilyIndex;
    uint32_t dstQueueFamilyIndex;
    VkImage image;
    VkImageSubresourceRange subresourceRange;
} VkImageMemoryBarrier;
```

What's interesting is that the **newLayout** and the **oldLayout** happens inbetween making the memory available and visible. 

The rules state that because we are doing a read/write operation the image must be available before the transition takes place. After the transition the memory layout is transitioned to be made **available** again BUT not **visible**. This transition happens on the L2 cache so the actual transitioning process is only for the L2 cache.

What this means it that if you have memory that's only in L1 cache and you try to transfer it from one type to another it wont work. If you later want to use this transitioned memory with an action you need to move the memory to be **visible** in L1 cache for work.
# References
##### Main Notes
[[How GPU memory works]]
#### Source Notes
[[Vulkan synchronisation explained]]