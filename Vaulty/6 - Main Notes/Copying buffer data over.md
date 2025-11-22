2025-11-18 08:56
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Copying buffer data over

You may want to turn this into a helper function. I have a followed the steps [[Creating a transfer queue for buffer copying]] which is different frm the tutorial code.

The first thing you need to do is create a command buffer

```c++
VkCommandBufferAllocateInfo allocateInfo{};
allocateInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO;
allocateInfo.level = VK_COMMAND_BUFFER_LEVEL_PRIMARY;
allocateInfo.commandPool = transferCommandPool;
allocateInfo.commandBufferCount = 1;

VkCommandBuffer commandBuffer;
vkAllocateCommandBuffers(device, &allocateInfo, &commandBuffer);
```

Next you'll want to record to it.

```c++
VkCommandBufferBeginInfo beginInfo{};
beginInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO;
beginInfo.flags = VK_COMMAND_BUFFER_USAGE_ONE_TIME_SUBMIT_BIT;

vkBeginCommandBuffer(commandBuffer, &beginInfo);

VkBufferCopy copyRegion{};
copyRegion.size = size;
vkCmdCopyBuffer(commandBuffer, srcBuffer, dstBuffer, 1, &copyRegion);

vkEndCommandBuffer(commandBuffer);
```

We want to begin the recording with **VK_COMMAND_BUFFER_USAGE_ONE_TIME_SUBMIT_BIT** as we are only gonna use this once. 

Next you want to record the **vkCmdCopyBuffer**. This needs to take a **srcBuffer** that has the TRANSFER_SRC_BIT as it's usage and the TRANSFER_DST_BIT for the usage for the other bit. For the destination you also need to OR the other usage flags together so we can actually use the buffer for something.

For the case of copying data into a vertex buffer you'll want to the usage like this:

The **VkBufferCopy** just sets up how much of the source buffer you're copying over to the destination.

Then you submit it
```c++
VkSubmitInfo submitInfo{};
submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;
submitInfo.commandBufferCount = 1;
submitInfo.pCommandBuffers = &commandBuffer;

vkQueueSubmit(transferQueue, 1, &submitInfo, VK_NULL_HANDLE);
vkQueueWaitIdle(transferQueue);
```

The faster way of waiting things to execute on the GPU is to wait for fence with **vkWaitForFences** this. Waiting for vkWaitForFences would allow you to stack multiple command buffer up to use the queue in parallel. This however requires more work and for 1 buffer transfer not even at run time **vkQueueWaitIdle** which waits until the queue is idling is good enough.

```c++
vkFreeCommandBuffers(device, transferCommandPool, 1, &commandBuffer);
```

Finally you've got to free the command buffer.
# References
##### Main Notes
[[Creating a transfer queue for buffer copying]]
#### Source Notes
[[Vertex buffer]]