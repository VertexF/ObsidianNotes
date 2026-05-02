2026-04-29 12:09
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# How memory layout transitions work

Just to start apart according to this [blog](https://themaister.net/blog/2019/08/14/yet-another-blog-explaining-vulkan-synchronization/) VkImage**Buffer**Barrier aren't interesting and no GPU really cares about them.

When you're working `VkImageMemoryBarrier` you have to change the image layout at some point. 

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

The rules state that memory that this is read/write operation and because of that the image must be available before the transition takes place. After the transition the memory is layout transition the memory is made **available** again BUT not **visible**. This transition happens on the L2 cache so the actual transitioning process is on the L2.
# References
##### Main Notes
[[How GPU memory works]]
#### Source Notes
[[Vulkan synchronisation explained]]