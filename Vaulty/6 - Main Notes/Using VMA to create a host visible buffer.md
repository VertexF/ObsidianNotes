2026-07-25 09:40
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Using VMA to create a host visible buffer

```c++
VmaAllocationCreateInfo memoryInfo{};
memoryInfo.flags = VMA_ALLOCATION_CREATE_HOST_ACCESS_SEQUENTIAL_WRITE_BIT | VMA_ALLOCATION_CREATE_HOST_ACCESS_ALLOW_TRANSFER_INSTEAD_BIT | VMA_ALLOCATION_CREATE_MAPPED_BIT;
memoryInfo.usage = VMA_MEMORY_USAGE_AUTO;

VmaAllocationInfo allocationInfo{};
check(vmaCreateBuffer(VMAAllocator, &bufferInfo, &memoryInfo, &buffer->vkBuffer, &buffer->vmaAllocation, &allocationInfo));
```

Using **VMA_MEMORY_USAGE_AUTO** will allow VMA to select the best piece of memory for the process. With the values **VMA_ALLOCATION_CREATE_HOST_ACCESS_SEQUENTIAL_WRITE_BIT** and **VMA_ALLOCATION_CREATE_HOST_ACCESS_ALLOW_TRANSFER_INSTEAD_BIT** both together make the things host visible.

We also add the **VMA_ALLOCATION_CREATE_MAPPED_BIT** flag which allows us to use vmaMapMemory and vmaUnmapMemory which are like the vkMapMemory and vkUnmapMemory but we are using the VMA allocator instead of our own + it's doing a lot more heavy lifting.

```c++
//NOTE we are doing some aliasing + allocating an index and vertex buffer here.
vmaMapMemory(allocator, vBufferAllocation, &bufferPtr);
memcpy(bufferPtr, vertices.data(), vBufSize);
memcpy(((char*)bufferPtr) + vBufSize, indices.data(), iBufSize);
vmaUnmapMemory(allocator, vBufferAllocation);
```

A simplier approach would be something like this.
```c++
vmaCopyMemoryToAllocation(gpu->VMAAllocator, &globalSceneData, globalSceneBuffer->vmaAllocation, 0, sizeof(UniformData));
```

This is the same thing as the one with `memcpy` but VMA itself does the `vmaMapMemory`. The arguments go
1) The VMA allocator you set up with [[Setting up VMA]]
2) A pointer to the buffer in memory.
3) The buffers allocator which is the `VmaAllocation`stuff we in the first code snippet.
4) The offset into the buffer.
5) The size of the memory to copy.

No matter how you allocate buffers you still need to make buffers that match the frames in flight of your application if the buffer is being updated at run time with [[CPU updating coherent buffers]]
# References
##### Main Notes
[[Hidden issues with VMA host visible buffers]]
[[CPU updating coherent buffers]]
[[Using VMA to create a GPU buffer]]
[[Setting up VMA]]
#### Source Notes
[[How to vulkan]]