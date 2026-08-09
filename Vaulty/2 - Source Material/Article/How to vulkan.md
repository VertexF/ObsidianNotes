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
##### Loading textures
The KTX file format is by Khronos. This is a format that's native to the GPU so there is no need to convert to a different type. It support mip mapping 3D texture and cubemaps. One tool for creating KTX image files is [PVRTexTool](https://developer.imaginationtech.com/solutions/pvrtextool/). This tool allows you to create cubemaps, IBL environment textures and add mip maps to texture.

There are 2 version ktx within the same library, version 1 is getting relatively old here. The two different formats for the actual files themselves are `.ktx` for version 1 and `ktx2` for version 2. If you can, use version 2. This kind all works like Vulkan version 2 of some function. 

With the ktx library you'll be able to load things from disk like this:
```c++
//version 1
ktxTexture* kTexture = nullptr;
constexpr const char* path = "path/to/file.ktx";
ktxTexture_createFromNamedFile(path, KTX_TEXTURE_CREATE_LOAD_IMAGE_DATA_BIT, &kTexture);

//version2
ktxTexture2* kTexture = nullptr;
constexpr const char* path = "path/to/file.ktx2";
ktxTexture2_createFromNamedFile(path, KTX_TEXTURE_CREATE_LOAD_IMAGE_DATA_BIT, &kTexture);
```

When you're using OpenGL and you request RGB instead RGBA it often secretly added a alpha channel to your file format. However, if you try to select this type within Vulkan you end ACTUALLY selecting RGB which isn't widely support so just RGBA.

Everything after this point is basically the same as using stb_image with a couple a changes. 

```c++
//Loaded kTexture here.

VkImageCreateInfo imageInfo{};
imageInfo.sType = VK_STRUCTURE_TYPE_IMAGE_CREATE_INFO;
imageInfo.flags = creation.layerCount == 1 ? 0 : VK_IMAGE_CREATE_CUBE_COMPATIBLE_BIT;
imageInfo.format = ktxTexture2_GetVkFormat(kTexture);
imageInfo.usage = VK_IMAGE_USAGE_TRANSFER_DST_BIT | VK_IMAGE_USAGE_SAMPLED_BIT;
imageInfo.imageType = VK_IMAGE_TYPE_2D;
imageInfo.extent.width  = kTexture->baseWidth;
imageInfo.extent.height = kTexture->baseHeight;
imageInfo.extent.depth = 1;
imageInfo.mipLevels = kTexture->numLevels;
imageInfo.arrayLayers = 1;
imageInfo.samples = VK_SAMPLE_COUNT_1_BIT;
imageInfo.tiling = VK_IMAGE_TILING_OPTIMAL;
imageInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;
imageInfo.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;

VmaAllocationCreateInfo memoryInfo{};
memoryInfo.flags = VMA_ALLOCATION_CREATE_DEDICATED_MEMORY_BIT;
memoryInfo.usage = VMA_MEMORY_USAGE_AUTO;

check(vmaCreateImage(gpu.VMAAllocator, &imageInfo, &memoryInfo, &texture->vkImage, &texture->vmaAllocation, nullptr));
```

Of course you want create an image view afterward for texture that would be done like this.

```c++
VkImageViewCreateInfo info{};
info.sType = VK_STRUCTURE_TYPE_IMAGE_VIEW_CREATE_INFO;
info.image = texture->vkImage;
info.viewType = VK_IMAGE_TYPE_2D;
info.format = imageInfo.format;
info.subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
info.subresourceRange.levelCount = kTexture->numLevels;
info.subresourceRange.layerCount = creation.layerCount;

check(vkCreateImageView(gpu.vulkanDevice, &info, gpu.vulkanAllocationCallbacks, &texture->vkImageView));
```

We of course need to do a staging to allow us to upload our textures with a optimal tiling which we have no way of knowing what type that is. In the void engine you just stick it in a queue and update at the end of the frame.

```c++
//Create stating buffer
VkBufferCreateInfo bufferInfo{};
bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
bufferInfo.usage = VK_BUFFER_USAGE_TRANSFER_SRC_BIT;

uint32_t totalBufferSize = (uint32_t)kTexture->dataSize;
uint32_t imageSize = (kTexture->baseWidth * kTexture->baseHeight * 4);
bufferInfo.size = totalBufferSize;

VmaAllocationCreateInfo memoryInfo{};
memoryInfo.flags = VMA_ALLOCATION_CREATE_HOST_ACCESS_SEQUENTIAL_WRITE_BIT;
memoryInfo.usage = VMA_MEMORY_USAGE_AUTO;

VmaAllocationInfo allocationInfo{};
VmaAllocation stagingAllocation;
VkBuffer stagingBuffer;
check(vmaCreateBuffer(VMAAllocator, &bufferInfo, &memoryInfo, &stagingBuffer, &stagingAllocation, &allocationInfo));
```

