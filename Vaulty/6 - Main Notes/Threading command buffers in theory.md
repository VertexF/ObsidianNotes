2025-12-23 19:47
Status: #baby 
Tags: [[vulkan]] [[vulkan command buffers]]
# Threading command buffers in theory

To be able to use command buffer in different threads, you'll need a command buffer and command pool per-thread and make sure that that each thread only uses the command buffer and pool they created. If you do that, then you can submit either command buffer is any thread.

Saying that **vkQueueSubmit** is not thread-safe. You can only have 1 thread push the command on a given queue at a time. It's common for a background thread to have things submitted and wait on this submission which is pretty expensive, while the main render thread goes on and continues executing.
# References
##### Main Notes
[[Setting up the command pool]]
[[Setting up the command buffers]]
#### Source Notes
[[VulkanDev]]