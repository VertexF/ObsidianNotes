2025-11-19 19:48
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Making vertex and index buffer faster with aliasing

In the example of [[Creating the vertex buffer with staging buffers]] and [[Creating the index buffer with staging buffers]] we have used a vertex buffer, allocated memory to it, bound it and the same for the index buffer. The problem is we are repeating the allocation for each buffer. 

Vulkan allows you to offset into buffers for different usages. Why not do that for the vertex and index buffer? This reduces expensive CPU allocations and increases GPU cache locality. With this method there is more for the programmer to keep track off, you'll have to keep track of the offsets and make things at set up and draw time both match.

Set up similar as before

```c++
VkDeviceSize vertexbufferSize = sizeof(vertices[0]) * vertices.size();
VkDeviceSize indexBufferSize = sizeof(indices[0]) * indices.size();
VkDeviceSize totalBufferSize = vertexbufferSize + indexBufferSize;

VkBuffer stagingBuffer;
VkDeviceMemory stagingBufferMemory;

createBuffer(totalBufferSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT,
VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, stagingBuffer, stagingBufferMemory);

void* data;
vkMapMemory(device, stagingBufferMemory, 0, vertexbufferSize, 0, &data);
memcpy(data, vertices.data(), size_t(vertexbufferSize));
vkUnmapMemory(device, stagingBufferMemory);

void* indexData;
vkMapMemory(device, stagingBufferMemory, vertexbufferSize, indexBufferSize, 0, &indexData);
memcpy(indexData, indices.data(), size_t(indexBufferSize));
vkUnmapMemory(device, stagingBufferMemory);

VkBuffer modelBuffer;
VkDeviceMemory modelBufferMemory;

createBuffer(totalBufferSize, VK_BUFFER_USAGE_TRANSFER_DST_BIT | VK_BUFFER_USAGE_VERTEX_BUFFER_BIT | VK_BUFFER_USAGE_INDEX_BUFFER_BIT,
VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, modelBuffer, modelBufferMemory);

copyBuffer(stagingBuffer, modelBuffer, totalBufferSize);

vkDestroyBuffer(device, stagingBuffer, nullptr);
vkFreeMemory(device, stagingBufferMemory, nullptr);
```


The we allocate 1 staging buffer for everything which takes in the entire size off the buffer with the variable **totalBufferSize**.

The next difference is we actually use the offset variables when allocating memory into the buffers. You can see that **vkMapMemory** third argument **vertexbufferSize** we are putting the index buffer after the vertex data in the staging buffers memory. Ideally you would reduce this even further with by having the index data and the vertex attributes in one container.

Finally we when we create the **modelBuffer** we make it a VK_BUFFER_USAGE_VERTEX_BUFFER_BIT | VK_BUFFER_USAGE_INDEX_BUFFER_BIT for both types.

Lets use these then
```c++
VkBuffer vertexBuffers[] = { modelBuffer };
VkDeviceSize offsets[] = { 0 };
vkCmdBindVertexBuffers(commandBuffers[currentFrame], 0, 1, vertexBuffers, offsets);
vkCmdBindIndexBuffer(commandBuffers[currentFrame], modelBuffer, vertexbufferSize, VK_INDEX_TYPE_UINT16);
```

This is pretty much the same but we know that the vertex buffer data is first so there is an offset of 0. However when we bind the index buffer we offset the buffer by the size of the vertex buffer with **vertexbufferSize** to make sure we read all the indices.

This is the simplest case, so if you're loading in model data please be aware of the buffer offsetting while doing this.
# References
##### Main Notes
[[Creating the vertex buffer with staging buffers]] 
[[Creating the index buffer with staging buffers]] 
[[Making a basic createBuffer function]]
[[Drawing with an index buffer]]
[[Binding the vertex buffer and drawing]]
#### Source Notes
[[Vertex buffer]]
