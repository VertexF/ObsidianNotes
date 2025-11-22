2025-11-18 08:47
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Creating a transfer queue for buffer copying

When you want to transfer data from one buffer to another you'll need to do that with a command on a command buffer. Meaning you'll need a queue that supports the transfer commands. You can just use the graphics queue as these support transfer commands. I didn't I made this work on it's own transfer queue.

To be able to do this you need to follow these steps.
1) You need to look for a queue that isn't VK_QUEUE_GRAPHICS_BIT and just a VK_QUEUE_TRANSFER_BIT. This is done as part of [[Checking for family queue suitability]]. So you should at least two different queue families indices. 
2) You need to make another VkQueue [[Creating vulkan queues]] which is part of the logical device creation with the queue family index.
3) Next you'll need to make a seperate VkCommandPool that is created with the transfer queue families index.  [[Setting up the command pool]] You want to create this just **ONCE** like the other one used for graphics commands.
4) Then you can create a command buffer that runs transfer commands outside of the "main" queue. [[Setting up the command buffers]] 

Note you don't have to make the buffers that are used in different queue **.sharingMode** VK_SHARING_MODE_CONCURRENT. 

You can see this all coming together in the [[Copying buffer data over]]
# References
##### Main Notes
[[Checking for family queue suitability]]
[[Creating vulkan queues]]
[[Setting up the command pool]]
[[Setting up the command buffers]]
[[Copying buffer data over]]
#### Source Notes
[[Vertex buffer]]