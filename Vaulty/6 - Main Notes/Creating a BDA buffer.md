2026-02-24 11:38
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]] [[buffer device address]]
# Creating a BDA buffer

Firstly you need to make sure the VMA knows how to allocate buffer with the Buffer Device Address set up with an extra flag in the creation struct of the creation struct.

```c++
VmaAllocatorCreateInfo allocatorInfo{};
allocatorInfo.flags = VMA_ALLOCATOR_CREATE_BUFFER_DEVICE_ADDRESS_BIT;
allocatorInfo.physicalDevice = vulkanPhysicalDevice;
allocatorInfo.device = vulkanDevice;
allocatorInfo.instance = vulkanInstance;
allocatorInfo.pVulkanFunctions = &vkFunctions;

vmaCreateAllocator(&allocatorInfo, &VMAAllocator);
```

Note the **.flag** has **VMA_ALLOCATOR_CREATE_BUFFER_DEVICE_ADDRESS_BIT**. This doesn't seem to affect non-BDA buffers being allocated.

Next you have to create the buffer in exactly the same way with the addition of the buffer device address fetch and few extra flags.

```c++
BufferHandle GPUDevice::createBindlessBuffer(const BufferCreation& creation) 
{
    BufferHandle handle = { buffers.obtainResource() };
    if (handle.index == INVALID_INDEX)
    {
        return handle;
    }

    Buffer* buffer = accessBuffer(handle);

    VkBufferCreateInfo bufferInfo{};
    bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
    bufferInfo.usage = VK_BUFFER_USAGE_SHADER_DEVICE_ADDRESS_BIT | creation.typeFlags;
    bufferInfo.size = creation.size > 0 ? creation.size : 1;

    VmaAllocationCreateInfo memoryInfo{};
    memoryInfo.flags = VMA_ALLOCATION_CREATE_HOST_ACCESS_SEQUENTIAL_WRITE_BIT | VMA_ALLOCATION_CREATE_HOST_ACCESS_ALLOW_TRANSFER_INSTEAD_BIT | VMA_ALLOCATION_CREATE_MAPPED_BIT;
    memoryInfo.usage = VMA_MEMORY_USAGE_AUTO;

    VmaAllocationInfo allocationInfo{};
    vmaCreateBuffer(VMAAllocator, &bufferInfo, &memoryInfo, &buffer->vkBuffer, &buffer->vmaAllocation, &allocationInfo);

    if (creation.initialData)
    {
		memcpy(allocationInfo.pMappedData, creation.initialData, static_cast<size_t>(creation.size));
    }

    VkBufferDeviceAddressInfo bufferBDAInfo{};
    bufferBDAInfo.sType = VK_STRUCTURE_TYPE_BUFFER_DEVICE_ADDRESS_INFO;
    bufferBDAInfo.buffer = buffer->vkBuffer;

    buffer->bufferAddress = vkGetBufferDeviceAddress(vulkanDevice, &bufferBDAInfo);

    return handle;
}
```

In the **VkBufferCreateInfo** we have a buffer creation struct which has the addition of the **.usage** **VK_BUFFER_USAGE_SHADER_DEVICE_ADDRESS_BIT** we are going to tell vulkan this buffer is a buffer device address buffer.

Everything else is exactly the same expect that we now fetch the `bufferAddress` with the function **vkGetBufferDeviceAddress** and it's **VkBufferDeviceAddressInfo** info struct.  
# References
##### Main Notes
[[What is Buffer Device Address (BDA)]]
#### Source Notes
