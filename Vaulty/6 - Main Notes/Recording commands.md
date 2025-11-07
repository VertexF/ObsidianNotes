2025-11-04 19:21
Status: #baby 
Tags: [[vulkan]] [[vulkan command buffers]]
# Recording commands

To do this you'll have to have finished [[Setting up the command buffers]]. Once you have a command buffer you can record any command into the command **IF** the command and the VkQueue type match. 

Before you begine it's important to note that you'll need to reset the command before with **vkResetCommandBuffer**.

```c++
vkResetCommandBuffer(commandBuffer, 0);
```

Second argument takes a reset flag which either is set **VK_COMMAND_BUFFER_RESET_RELEASE_RESOURCES_BIT** or 0. If you set this the **REST_RELEASER_RESOURCES_BIT*** then all the memory that was used in the recording process will be released back into the command pool. 
If **0** then the command buffer **MAY** hold on to memory. For use, this doesn't matter really so it's 0.

When you want to begin recording you need the **VkCommandBufferBeginInfo** struct which begins and ends the command buffer recording process. 

This is how things are began, **vkBeginCommandBuffer** clears the previously recorded commands, you can previously append more commands. 
```c++
VkCommandBufferBeginInfo beginInfo{};
beginInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO;
beginInfo.flags = 0;
beginInfo.pInheritanceInfo = 0;

if (vkBeginCommandBuffer(commandBuffer, &beginInfo) != VK_SUCCESS) 
{
	printf("Failed to begin recording command buffer.");
}
```

The **.flag** has a couple different settings you might want to consider. 
- **VK_COMMAND_BUFFER_USAGE_ONE_TIME_SUBMIT** The command buffer will be rerecorded right after exection.
- **VK_COMMAND_BUFFER_RENDER_PASS_CONTINUE_BIT** This is used for the secondary command buffer and it says it will be used entirely within a single render pass.
- **VK_COMMAND_BUFFER_USAGE_SIMULTANEOUS_USE_BIT** This is for a command buffer that is resubmitted while it is already pending for execution.

**.pInheritanceInfo** is used for secondary command buffer, this states which primary command buffer it inherits from.

Between ending and beginning the command buffer you can write any command which all start with **vkCmd** to the buffer. (Assuming the command match the VkQueue you set the VkCommandPool up with)

To end a command buffer you run this.
```c++
if (vkEndCommandBuffer(commandBuffer) != VK_SUCCESS) 
{
	printf("Failed to record a command buffer");
}
```

This takes the command buffer out of a recording state and now can be submitted. After **vkEndCommandBuffer** you can't record any more commands.
# References
##### Main Notes
[[Using a render pass]]
[[What are command buffers]]
[[Setting up the command pool]]
[[Setting up the command buffers]]
#### Source Notes
[[Vulkan-Tutorial]]