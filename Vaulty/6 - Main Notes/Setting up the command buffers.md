2025-11-04 19:13
Status: #baby 
Tags: [[vulkan]] [[vulkan command buffers]]
# Setting up the command buffers

When you create a command buffer you need to have finshed [[Setting up the command pool]]. It's important to note that when you create a command buffer they are deleted with the command pool.

To create the command buffers you need to allocate them from the command pool you should have set up. 

```c++
VkCommandBufferAllocateInfo allocateInfo{};
allocateInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO;
allocateInfo.commandPool = commandPool;
allocateInfo.level = VK_COMMAND_BUFFER_LEVEL_PRIMARY;
allocateInfo.commandBufferCount = 1;

VkCommandBuffer commandBuffer;
if (vkAllocateCommandBuffers(device, &allocateInfo, &commandBuffer)) 
{
	printf("Failed to create a command buffer.");
}
```

The creation struct has levels you need to set up, primary and secondary.
-  **VK_COMMAND_BUFFER_LEVEL_PRIMARY** These **CAN** be submitted to a queue for execution but cannot call other command buffers.
- **VK_COMMAND_BUFFER_LEVEL_SECONDARY** These **CAN'T** be submitted directly, but can be called from primary command buffers.

Secondary command buffer usage helps with command reuse which can help with performance. They can't however they can't run command that require things to be submitted to a queue directly.
# References
##### Main Notes
[[Recording commands]]
[[Setting up the command pool]]
#### Source Notes
[[Vulkan-Tutorial]]