2026-07-25 10:55
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]]
# Creating a texture with KTX file

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
# References
##### Main Notes
[[Setting up the KTX library]]
[[What is KTX]]
[[Using VMA to create a host visible buffer]]
[[Creating a VkImage for texture mapping]]
[[Transitioning image memory for texture mapping]]
[[Preparing the texture image]]
#### Source Notes
[[How to vulkan]]