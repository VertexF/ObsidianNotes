2026-02-11 09:20
Status: #baby 
Tags: [[vulkan]] [[vulkan command buffers]]
# Command buffer states

When you first create a command buffer it starts in a **Ready** state. Once you run a **vkBeginCommandBuffer** you put the command buffer in a **Record** state. Once you run **vkEndCommandBuffer** the command buffer is an **Executable** state. 

After submitting a command buffer it does into a **Pending** state. It's in a pending state because you need to wait for the work to be done then you reset it, with **vkResetCommandBuffer**. 

This is why I've set things up that for each frame we are going to use within a swapchain, we have command buffers for each image. 
# References
##### Main Notes
[[What are command buffers]]
[[Submitting the command buffer for rendering]]
#### Source Notes
[[VulkanDev]]