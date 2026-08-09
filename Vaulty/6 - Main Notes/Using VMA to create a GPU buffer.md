2026-07-25 10:48
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Using VMA to create a GPU buffer

This is very similar to [[Using VMA to create a host visible buffer]] but the `VmaAllocationCreateInfo` requires some different settings here.

```c++
Buffer* buffer = accessBuffer(handle);
buffer->name = creation.name;
buffer->size = creation.size;
buffer->typeFlags = creation.typeFlags;
buffer->handle = handle;
buffer->globalOffset = 0;

VkBufferCreateInfo bufferInfo{};
bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
bufferInfo.usage = VK_BUFFER_USAGE_SHADER_DEVICE_ADDRESS_BIT | creation.typeFlags;
bufferInfo.size = creation.size > 0 ? creation.size : 1;

VmaAllocationCreateInfo memoryInfo{};
memoryInfo.usage = VMA_MEMORY_USAGE_AUTO_PREFER_DEVICE;
memoryInfo.requiredFlags = VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT;

VmaAllocationInfo allocationInfo{};
check(vmaCreateBuffer(VMAAllocator, &bufferInfo, &memoryInfo, &buffer->vkBuffer, &buffer->vmaAllocation, &allocationInfo));
```

Here I'm creating a buffer that I will access with a buffer device address but that's unimportant. To make a buffer with VMA be GPU exclusive you use `memoryInfo.requiredFlags` with `VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT` we leave alone the `memoryInfo.flags`.

You can either use `VMA_MEMORY_USAGE_AUTO_PREFER_DEVICE` or `VMA_MEMORY_USAGE_AUTO` for usage.
# References
##### Main Notes
[[Using VMA to create a host visible buffer]]
[[Setting up VMA]]
[[Setting up a vulkan buffer]]
#### Source Notes
