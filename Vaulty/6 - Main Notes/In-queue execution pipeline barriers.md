2026-04-27 13:17
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# In-queue execution pipeline barriers

An execution barrier is a pipeline barrier. A execution barrier is a subset of memory barriers.

```c++
void vkCmdPipelineBarrier(
VkCommandBuffer commandBuffer,
VkPipelineStageFlags srcStageMask,
VkPipelineStageFlags dstStageMask,
VkDependencyFlags dependencyFlags,
uint32_t memoryBarrierCount,
const VkMemoryBarrier* pMemoryBarriers,
uint32_t bufferMemoryBarrierCount,
const VkBufferMemoryBarrier* pBufferMemoryBarriers,
uint32_t imageMemoryBarrierCount,
const VkImageMemoryBarrier*pImageMemoryBarriers);
```

If we ignore the all the memory barrier flags in the function above you just have the **srcStageMask** and the **dstStageMask**. Which are only interested in execution order.

The **srcStageMask** and the **dstStageMask** are at the heart of synchronisation, in the command stream we have a before **srcStageMask** and **dstStageMask** for after.
# References
##### Main Notes
[[The pipeline stages]]
[[What are pipeline barriers]]
[[The difference between source and destination stage mask]]
#### Source Notes
[[Vulkan synchronisation explained]]