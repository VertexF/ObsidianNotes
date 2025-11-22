2025-11-19 17:41
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Creating the index buffer with staging buffers

This will be basically the same as [[Creating the vertex buffer with staging buffers]] with like 2 differences

```c++
VkBuffer indexBuffer;
VkDeviceMemory indexBufferMemory;
VkBuffer stagingBuffer;
VkDeviceMemory stagingBufferMemory;

VkDeviceSize indexBufferSize = sizeof(indices[0]) * indices.size();

createBuffer(indexBufferSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT,
VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, stagingBuffer, stagingBufferMemory);

void* indexData;
vkMapMemory(device, stagingBufferMemory, 0, indexBufferSize, 0, &indexData);
memcpy(indexData, indices.data(), size_t(bufferSize));
vkUnmapMemory(device, stagingBufferMemory);

createBuffer(indexBufferSize, VK_BUFFER_USAGE_TRANSFER_DST_BIT | VK_BUFFER_USAGE_INDEX_BUFFER_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, indexBuffer, indexBufferMemory);

copyBuffer(stagingBuffer, indexBuffer, indexBufferSize);

vkDestroyBuffer(device, stagingBuffer, nullptr);
vkFreeMemory(device, stagingBufferMemory, nullptr);
```

The only differences here are the **indexBufferSize** which is equal to the **indices** which is were the uint16_t list of numbers live. When we create the index buffers with **createBuffer** we use an **VK_BUFFER_USAGE_INDEX_BUFFER_BIT** 

Like the vertex buffer you need to destroy them too.

```c++
vkDestroyBuffer(device, indexBuffer, nullptr);
vkFreeMemory(device, indexBufferMemory, nullptr);
```
# References
##### Main Notes
[[Setting up the index buffer data]]
[[Making a basic createBuffer function]]
[[What is a index buffer]]
[[What are staging buffers]]
[[Creating a transfer queue for buffer copying]]
[[Setting up a staging buffer]]
[[Copying buffer data over]]
#### Source Notes
[[Vertex buffer]]