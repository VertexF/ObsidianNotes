2025-11-27 12:03
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]]
# Creating a VkImage for texture mapping

We need to convert the staging buffer to an image, you can use the buffer raw but it's slower. If you add the pixel data to the image the pixels are called texels which is pixels with corrdinates.

So lets get started
```c++
VkImage textureImage;
VkDeviceMemory textureImageMemory;

VkImageCreateInfo imageInfo{};
imageInfo.sType = VK_STRUCTURE_TYPE_IMAGE_CREATE_INFO;
imageInfo.imageType = VK_IMAGE_TYPE_2D;
imageInfo.extent.width = uint32_t(texWidth);
imageInfo.extent.height = uint32_t(texHeight);
imageInfo.extent.depth = 1;
imageInfo.mipLevels = 1;
imageInfo.arrayLayers = 1;
```

We need to first tell vulkan first what image type the texels need to be addressed. 
- **VK_IMAGE_TYPE_1D** is used for storing data in array or a gradient
- **VK_IMAGE_TYPE_2D** is used most for textures
- **VK_IMAGE_TYPE_3D** is used to store voxel volumes.

The **.extents** specifies the dimensions of the image. Meaning that the extents will be different depending on the **.imageTypes**. With 2D textures we importantly need a depth of 1 which is the Z value in the space for our image, the width and height are the X and Y. Now vulkan knows how to map each pixel into texel space.

We will skipping mip levels and we are not making an array so we are gonna skip these with a value of one.

```c++
imageInfo.format = VK_FORMAT_R8G8B8A8_SRGB;
```

The format is important to match the same data as the pixel data type we got stbi_image or the copy will fail.

```c++
imageInfo.tiling = VK_IMAGE_TILING_OPTIMAL;
```

The **.tiling** two values
- **VK_IMAGE_TILING_LINEAR** The texel are laid out in a row-major order like the stbi_image layout of the pixel data.
- **VK_IMAGE_TILING_OPTIMAL** The texels are laid out in a implementation defined order for optimal access.

You can't change the tiling type at run time. Meaning if you don't plan to change it you want **VK_IMAGE_TILING_OPTIMAL** if you want to change the tiling mode later you need **VK_IMAGE_TILING_LINEAR**.

```c++
imageInfo.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
```

There are only 2 values for the **.initalLayout**
1) **VK_IMAGE_LAYOUT_UNDEFINED** This tells us that the very first memory transition isn't used by the GPU and will discard the texels.
2) **VK_IMAGE_LAYOUT_PREINITIALIZED** This tells us that the memory isn't being used by the GPU. However we **ARE** preseving the texels.

You would want **VK_IMAGE_LAYOUT_PREINITIALIZED** when you want to preseve what the data. An example, would be if you're set the **.tiling** to linear. If you did that you would want to upload the texel data to it and transition it to be the source image without losing data. If you're going to transition the image first to be a destination and then copy the texel data to it from a buffer, like we've done then you don't want to do this.

```c++
imageInfo.usage = VK_IMAGE_USAGE_TRANSFER_DST_BIT | VK_IMAGE_USAGE_SAMPLED_BIT;
```

This is just like a buffer usage. Here we set the image to be able to recieve a transfer with destination, and setting the **VK_IMAGE_USAGE_SAMPLED_BIT** means we can put that in the shader.

```c++
imageInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;
```

The operation isn't going across queue so exclusive is good.

```c++
imageInfo.samples = VK_SAMPLE_COUNT_1_BIT;
imageInfo.flags = 0;
```

We want to have 1_BIT to match the rasterizer graphics pipeline which is 1. The flag is used for sparse textures, this can be used for voxel textures. For example, if you have 3D textures are expensive to allocate so with the sparse data you can skip a lot of the "air" part of the texture.

```c++
if (vkCreateImage(device, &imageInfo, nullptr, &textureImage) != VK_SUCCESS) 
{
    printf("Failed to create image.");
}
```

Here you really should check if **VK_FORMAT_R8G8B8A8_SRGB** is supported, but it's so commonly supported that it's really not worth checking. You would have to do a lot of conversation if you use a different format than this anyway.

```c++
VkMemoryRequirements memRequirements;
vkGetImageMemoryRequirements(device, textureImage, &memRequirements);

VkMemoryAllocateInfo allocateInfo{};
allocateInfo.sType = VK_STRUCTURE_TYPE_MEMORY_ALLOCATE_INFO;
allocateInfo.allocationSize = memRequirements.size;
allocateInfo.memoryTypeIndex = findMemoryType(memRequirements.memoryTypeBits, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT);

if (vkAllocateMemory(device, &allocateInfo, nullptr, &textureImageMemory) != VK_SUCCESS) 
{
    printf("Failed to allocate image memory");
}

vkBindImageMemory(device, textureImage, textureImageMemory, 0);
```

You then set the memory exactly the same way as when you create the buffers [[Setting up the allocating buffer memory]] The only difference is I've abstracted the **.memoryTypeIndex** into it's own function.

This was all encapsulated in a helper function called **copyBufferToImage** as it's going to be used in different places.
# References
##### Main Notes
[[Setting up the allocating buffer memory]]
[[Introduction to texture mapping]]
[[Loading an image file]]
#### Source Notes
[[Texture mapping]]
