2026-07-25 09:52
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Hidden issues with VMA host visible buffers

When you allocate buffers like this in the normal way like this. There are hidden dangers that can trip you up, because validation as the time of writing this will NOT pick up mistaken usage.
```c++
VmaAllocationCreateInfo memoryInfo{};
memoryInfo.flags = VMA_ALLOCATION_CREATE_HOST_ACCESS_SEQUENTIAL_WRITE_BIT | VMA_ALLOCATION_CREATE_HOST_ACCESS_ALLOW_TRANSFER_INSTEAD_BIT | VMA_ALLOCATION_CREATE_MAPPED_BIT;
memoryInfo.usage = VMA_MEMORY_USAGE_AUTO;

VmaAllocationInfo allocationInfo{};
check(vmaCreateBuffer(VMAAllocator, &bufferInfo, &memoryInfo, &buffer->vkBuffer, &buffer->vmaAllocation, &allocationInfo));
```

I've personally limited success with storing the pointer of a mapped buffer. So I personally don't use `VMA_ALLOCATION_CREATE_MAPPED_BIT` if I can avoid it at all. I just use `vmaCopyMemoryToAllocation` instead of keeping a `void*` to the buffer data.

If you only plan to update a buffer on the CPU and **NOT** you can just the flag `VMA_ALLOCATION_CREATE_HOST_ACCESS_SEQUENTIAL_WRITE_BIT` however you have to be careful not to accidently read this buffer because it's very slow and likely unstable.

For example this is a bad
```c++
VkBufferCreateInfo bufferInfo{};
bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
bufferInfo.size = sizeof(vertices[0]) * vertices.size();
bufferInfo.usage = VK_BUFFER_USAGE_STORAGE_BUFFER_BIT;
bufferInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;

Buffer* buffer = accessBuffer(handle);
buffer->name = "buffer";
buffer->size = sizeof(vertices[0]) * vertices.size();;
buffer->typeFlags = creation.typeFlags;
buffer->handle = handle;
buffer->globalOffset = 0;

VmaAllocationCreateInfo memoryInfo{};
memoryInfo.flags = VMA_ALLOCATION_CREATE_HOST_ACCESS_ALLOW_TRANSFER_INSTEAD_BIT;
memoryInfo.usage = VMA_MEMORY_USAGE_AUTO;

VmaAllocationInfo allocationInfo{};
check(vmaCreateBuffer(VMAAllocator, &bufferInfo, &memoryInfo, &buffer->vkBuffer, &buffer->vmaAllocation, &allocationInfo));

//Render loop
while(1)
{
	//BAD!!!
	vertices[0].x += 10;
}
```

In the render loop we are reading the buffer here making things slow. You the need `VMA_ALLOCATION_CREATE_HOST_ACCESS_RANDOM_BIT` if you want to randomly access elements in a buffer. Having read and write access is still slower so if you can avoid and just have straight writes then your chillin.
# References
##### Main Notes
[[Using VMA to create a host visible buffer]]
[[Setting up a vulkan buffer]]
#### Source Notes
