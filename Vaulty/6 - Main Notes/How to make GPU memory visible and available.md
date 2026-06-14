2026-04-27 13:55
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# How to make GPU memory visible and available

Below is an example of how to take L1 cache and make it **available** to the L2 cache for a colour attachment.
```c++
VkMemoryBarrier2 memoryBarrier;
memoryBarrier.srcStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT;
memoryBarrier.srcAccessMask = VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT;
```

In the example were the L1 cache needs to update the L2 cache we need to finish the `VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT` . The `VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT` moves the L1 cache into L2 cache making it **available** with `COLOR_ATTACHMENT_WRITE_BIT`. For different situations you would need to change the memory type to something else.

Using the destination flags we are moving the L2 to L1 making the memory **visible**
```c++
memoryBarrier.dstStageMask = VK_PIPELINE_STAGE_COLOR_FRAGMENT_SHADER_BIT;
memoryBarrier.dstAccessMask = VK_ACCESS_COLOR_SHADER_READ_BIT;
```

For example, lets say we have `vkCmd1` that writes out colour memory to it's L1 cache. `vkCmd2` wants to do something after with that memory `vkCmd1` has to finish. So we need to pull the memory from L2 cache to the L1 cache.

The source access moves the memory from `vkCmd1`'s L1 cache to L2, with the destination mask telling `vkCmd2` to transfer memory from L2 to it's own L1 making it **visible** so it can complete it's operation in it's fragment shader stage.
# References
##### Main Notes
[[How GPU memory works]]
[[What are the access masks]]
[[The difference between source and destination stage mask]]
[[Making non-global pipeline barriers]]
[[In-queue execution pipeline barriers]]
[[The pipeline stages]]
[[Vulkan command categories]]
[[Implicit synchronisation guarantees]]
[[Basic queue synchronisation]]
[[Vulkan command overlap]]
#### Source Notes
[[Vulkan synchronisation explained]]