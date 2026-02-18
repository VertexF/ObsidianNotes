# Reference https://www.howtovulkan.com/#setting-up-vma

##### Setting up VMA

To use VMA you need to create the VMA allocator with a creation struct

```c++
VmaAllocatorCreateInfo allocatorInfo{};
allocatorInfo.flags = VMA_ALLOCATOR_CREATE_BUFFER_DEVICE_ADDRESS_BIT;
allocatorInfo.physicalDevice = vulkanPhysicalDevice;
allocatorInfo.device = vulkanDevice;
allocatorInfo.instance = vulkanInstance;

result = vmaCreateAllocator(&allocatorInfo, &VMAAllocator);
check(result);
```

You need the device, instance and physical device. Then you can return the successfully create     **VmaAllocator**.

The flag **VMA_ALLOCATOR_CREATE_BUFFER_DEVICE_ADDRESS_BIT** is only needed if you have the bufferDeviceAddress feature available. 

That's pretty much all you need to do for the set up but you can set up common function that will be used with a **VmaFunction** set up.

```c++
VmaVulkanFunctions vkFunctions{};
vkFunctions.vkGetInstanceProcAddr = vkGetInstanceProcAddr;
vkFunctions.vkGetDeviceProcAddr = vkGetDeviceProcAddr;
vkFunctions.vkCreateImage = vkCreateImage;

VmaAllocatorCreateInfo allocatorInfo{};
allocatorInfo.pVulkanFunctions = &vkFunctions;
```

This set up VMA so it used in these common function calls. However, I don't use this so I haven't tested it.

##### Using VMA to create an image

Generally you would want to set up things like for images.
```c++
VmaAllocationCreateInfo memoryInfo{};
memoryInfo.flags = VMA_ALLOCATION_CREATE_DEDICATED_MEMORY_BIT;
memoryInfo.usage = VMA_MEMORY_USAGE_AUTO;

check(vmaCreateImage(gpu.VMAAllocator, &imageInfo, &memoryInfo, &texture->vkImage, &texture->vmaAllocation, nullptr));
```

Using **VMA_MEMORY_USAGE_AUTO** will allow VMA to select the best piece of memory for the process. Also using the **VMA_ALLOCATION_CREATE_DEDICATED_MEMORY_BIT** allows for VMA to create a new allocator for big resources, this is good for large image attachments.
##### Using VMA to create a buffer

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

