2025-11-04 18:57
Status: #baby 
Tags: [[vulkan]] [[vulkan command buffers]]
# Setting up the command pool

When you want to create command buffers you need to do it from a command pool. These manage the memory of the command buffer that you allocate them from.

We have to set the family queue index for the queue we want to set the command buffers on. You also need to tell vulkan how you're going to use the command buffers by setting the **.flags**
```c++
VkCommandPoolCreateInfo poolCreateInfo{};
poolCreateInfo.sType = VK_STRUCTURE_TYPE_COMMAND_POOL_CREATE_INFO;
poolCreateInfo.flags = VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT;
poolCreateInfo.queueFamilyIndex = mainQueueFamilyIndex;
```

The flag can be in two states
- **VK_COMMAND_POOL_CREATE_TRANSIENT_BIT** This is for command that will be recorded very frequenctly, which means there is a tonne of allocation happening.
- **VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT** This means that the command buffer will be rerecorded individually. Without this flag the command buffer will be reset together in one go.

In our example, we are setting up the most typical set up for command pools. If you want something else please consider what you're doing.

After that you can create and destroy them like any other object.

```c++
VkCommandPool commandPool;
if (vkCreateCommandPool(device, &poolCreateInfo, nullptr, &commandPool))
{
	printf("Failed to create a command pool.");
}
```

Delete this at the end buffer the device gets deleted.
```c++
vkDestroyCommandPool(device, commandPool, nullptr);
```
# References
##### Main Notes
[[What are command buffers]]
[[Setting up the command buffers]]
[[What are queue families]]
[[Listing the queue families]]
[[Checking for family queue suitability]]
[[Setting up vulkan queues]]
[[Creating vulkan queues]]
#### Source Notes
[[Drawing a triangle]]