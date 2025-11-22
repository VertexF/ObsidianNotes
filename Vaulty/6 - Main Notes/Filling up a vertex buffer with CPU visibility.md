2025-11-16 16:56
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Filling up a vertex buffer with CPU visibility

```c++
void* data;
vkMapMemory(device, vertexBufferMemory, 0, bufferInfo.size, 0, &data);
memcpy(data, vertices.data(), size_t(bufferInfo.size));
vkUnmapMemory(device, vertexBufferMemory);
```

When using one buffer to load memory into the buffer you need to be aware that there are two ways of doing something. 
1) Is using the memory heap that is host coherent with VK_MEMORY_PROPERTY_HOST_COHERNET_BIT
2) Flushing the buffer with **vkFlushMappedMemoryRanges** when you're writing to a buffer and call **vkInvalidateMappedMemoryRanges** before reading the mapped memory.

You need to worry about this because the GPU doesn't guarantee when it will be uploaded because of caching. Correctly we use VK_MEMORY_PROPERTY_HOST_COHERNET_BIT which is slower. The only thing we can be sure about is that it will be available when we get to the **vkQueueSubmit**.
# References
##### Main Notes
[[Setting up the a buffer]]
[[Setting up the allocating buffer memory]]
#### Source Notes
[[Vertex buffer]]