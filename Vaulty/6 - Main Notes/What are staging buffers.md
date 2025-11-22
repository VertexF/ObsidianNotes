2025-11-18 14:18
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# What are staging buffers

Staging buffer are used as an intermediate buffer before the final buffer. They are used to upload data to and then we transfer the data to the final buffer. The reason you would want to do this if for speed. 

So the idea goes you want a buffer on the GPU to have the **VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT** which means it's local to the GPU, meaning two things. 1) It's faster and 2) The host can't see the memory. So you use a staging buffer that is visible to the host upload that, and then do a transfer command with a command buffer to move it over to the buffer we care about.

After that we destroy and delete the staging buffer once it's down.
# References
##### Main Notes
[[Creating a transfer queue for buffer copying]]
[[Setting up a staging buffer]]
#### Source Notes
[[Vertex buffer]]