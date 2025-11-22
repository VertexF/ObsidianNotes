2025-11-18 14:08
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Creating the vertex buffer with staging buffers

Before reading it's serously important you read the [[Setting up a staging buffer]] and [[Copying buffer data over]] notes if you're confused.

Here is a code snippet on how create the vertex buffer using staging buffers copy buffer.
```c++
createBuffer(bufferSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT, 
VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, stagingBuffer, stagingBufferMemory);

void* data;
vkMapMemory(device, stagingBufferMemory, 0, bufferSize, 0, &data);
memcpy(data, vertices.data(), size_t(bufferSize));
vkUnmapMemory(device, stagingBufferMemory);

VkBuffer vertexBuffer;
VkDeviceMemory vertexBufferMemory;

createBuffer(bufferSize, VK_BUFFER_USAGE_TRANSFER_DST_BIT | VK_BUFFER_USAGE_VERTEX_BUFFER_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, vertexBuffer, vertexBufferMemory);

copyBuffer(stagingBuffer, vertexBuffer, bufferSize);

vkDestroyBuffer(device, stagingBuffer, nullptr);
vkFreeMemory(device, stagingBufferMemory, nullptr);
```

1) We create the staging buffer
2) We load memory into the staging buffer
3) We create the vertex buffer
4) We copy the staging buffer data to the vertex buffer which happens on the GPU
5) We destroy the staging buffers

The **createBuffer** second and third arguments contains the 
2) memory usage
3) memory type/location. 

These sets up what the buffer does and where the buffer lives.

The pattern is pretty solid so this pattern could be used for other types of buffers that aren't vertex buffers.
# References
##### Main Notes
[[Setting up a staging buffer]]
[[Making a basic createBuffer function]]
[[Creating a transfer queue for buffer copying]]
[[Copying buffer data over]]
[[Setting up the allocating buffer memory]]
[[Filling up a vertex buffer with CPU visibility]]
[[Setting up the a buffer]]
#### Source Notes
[[Vertex buffer]]