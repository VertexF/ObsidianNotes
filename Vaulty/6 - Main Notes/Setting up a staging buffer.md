2025-11-18 14:14
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Setting up a staging buffer

It's important to note that you likely want to abstract creation of buffers into functions if you're planning on doing this. I'm also going to create a vertex buffer with a staging buffer the same patterns apply.

```c++
VkDeviceSize bufferSize = sizeof(vertices[0]) * vertices.size();

VkBuffer stagingBuffer;
VkDeviceMemory stagingBufferMemory;

createBuffer(bufferSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT, 
VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, stagingBuffer, stagingBufferMemory);

void* data;
vkMapMemory(device, stagingBufferMemory, 0, bufferSize, 0, &data);
memcpy(data, vertices.data(), size_t(bufferSize));
vkUnmapMemory(device, stagingBufferMemory);
```

Okay so the **createBuffer** is a help function that creates a buffer, selects the memory type and binds it. You can find more about that in the [[Setting up the a buffer]] and [[Allocating data to a buffer]] The arguments for this function are:
1) **VkDeviceSize** which takes in the size of the buffer. 
2) **VkBufferUsageFlags** This tells the function what the buffer is going to be used for. 
3) **VkMemoryPropertyFlags** This tell the function the memory type we are interested in. 
4) **VkBuffer&** is the output variable for the buffer we are going to use. 
5) **VkDeviceMemory&** is the output variable for the buffer **memory** we are going to use. 

The **VkBufferUsageFlags** are important to make this buffer transferable. The two values you want to know about for this are.
- **VK_BUFFER_USAGE_TRANSFER_SRC_BIT** Usage for when buffer are the source of the memory transfer operation.
- **VK_BUFFER_USAGE_TRANSFER_DST_BIT** Usage for when buffer are the destination of the memory transfer operation.

Remember since the **VkMemoryPropertyFlags** is host visible and coherent we can upload the memory into the buffer no problem.
# References
##### Main Notes
[[What are staging buffers]]
[[Setting up the a buffer]]
[[Allocating data to a buffer]] 
#### Source Notes
[[Vertex buffer]]