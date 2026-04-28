2026-04-27 13:55
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# How to make GPU memory visible and available

Lets say that in warp work has done has written some colour values out after being processed. This would happen at the `VK_PIPELINE_STAGE_COLOUR_ATTACHMENT_OUTPUT_BIT` and we would have the `VK_ACCESS_COLOUR_ATTACHMENT_WRITE_BIT` mask to represent what type of memory we have in the L1 cache.

There is a BIG problem. Now the L1 cache in that warp isn't the same as the L2 cache meaning it's not available to be used by other warps. This is why we both use a source and destination access to mask to make sure the memory transfer is completed before we move on with further execution.

To fix this you need to combined both stage and access mask together like this.
```c++
VkMemoryBarrier2 memoryBarrier;
memoryBarrier.srcStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT;
memoryBarrier.srcAccessMask = VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT;
```

In the example were the L1 cache needs to update the L2 cache we need to finish the `VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT` the `VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT` change needs to moved to the L2 cache making it visible. 

The destination access mask work with the destination stage mask to flag memory that needs to become visible for the destination stage before executing. 
```c++
memoryBarrier.dstStageMask = VK_PIPELINE_STAGE_COLOR_FRAGMENT_SHADER_BIT;
memoryBarrier.dstAccessMask = VK_ACCESS_COLOR_SHADER_READ_BIT;
```

For example, lets say we have `vkCmd1` that writes out colour memory to it's L1 cache. `vkCmd2` wants to do something after that `vkCmd1` has finish AND uses it's memory output in it's shader.

The source access moves the memory from `vkCmd1` L1 cache to L2, with the destination mask tells `vkCmd2` to transfer memory from L2 to it's L1 making it visible so it can complete it's operation in it's fragment shader stage.
# References
##### Main Notes
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