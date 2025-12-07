# Reference https://vulkan-tutorial.com/Texture_mapping/Images

Texture mapping is pretty complicated and requires good use of memory layout and transitions. The overall idea of adding a texture to some polygons is to:
- Create an image back by device memory
- Fill it with pixels from a file.
- Create an image sampler
- We'll use our descriptors we made to add to sample our colours from in the fragment shader.

Yes we have already talking about this in [[What are image views]] and [[Creating an image view]] but we need to do more than we did to handle textures. It's similar to seting up a vertex buffer honestly, you just need to make a staging buffer, then transfer over the staged pixles into our final image, ready to be used.

We also need to worry about memory layouts on top of this. We seen some of these before so here a quick list.
1) **VK_IMAGE_LAYOUT_PRESENT_SRC** optimal for prensentation.
2) **VK_IMAGE_LAYOUT_COLOUR_ATTACHMENT_OPTIMAL** optimal as an attachment for writing from fragment shader.
3) **VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL** optimal for being the source in transfer operations.
4) **VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL** optimal for being the destination in transfer operations.
5) **VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL** optimal for sampling from a shader.

When handling texture and images that come from that you'll need to also use pipeline barriers. We used this for example in [[Setting the synchronisation with the subpass dependencies]] 
##### Loading an image file
I'll be using the fantastic library for loading images the stb_image.h. So include that in your project and forget to add the **STB_IMAGE_IMPLEMENTATION** declaration before the include.
```c++
#define STB_IMAGE_IMPLEMENTATION
#include <stb_image.h>
```

I will be brief because I've done this before. This is how you load an image with stbi_image

```c++
//"Textures/frobert.png"
static void createTextureImage(const char* path, VkDeviceSize &bufferSize, stbi_uc*& pixels)
{
    int texWidth;
    int texHeight;
    int channles;

    pixels = stbi_load(path, &texWidth, &texHeight, &channles, STBI_rgb_alpha);
    bufferSize = texWidth * texHeight * 4;

    if (!pixels)
    {
        printf("Failed to load texture image.");
    }
}
```

With the two output variable we can now create staging buffer that load these pixels in to memory with a host visible staging buffer. [[Creating the vertex buffer with staging buffers]] explains this in more details

```c++
stbi_uc* pixels = nullptr;
VkDeviceSize imageSize;
createTextureImage("Textures/frobert.png", imageSize, pixels);

VkBuffer imageStagingBuffer;
VkDeviceMemory imageStagingBufferMemory;

createBuffer(imageSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, imageStagingBuffer, imageStagingBufferMemory);

void* imageData;
vkMapMemory(device, imageStagingBufferMemory, 0, imageSize, 0, &imageData);
memcpy(imageData, pixels, size_t(imageSize));
vkUnmapMemory(device, imageStagingBufferMemory);

stbi_image_free(pixels);
```

Don't forget to free the pixels once the memcpy has happend.
##### Creation of a VkImage for textures
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
2) **VK_IMAGE_LAYOUT_PREINITIALIZED** This tells us that the memory isn't being used by the GPU. However we **ARE** preversing the texels.

You would want **VK_IMAGE_LAYOUT_PREINITIALIZED** when you want to preseve what the data. An example, would be if you're set the **.tiling** to linear. If you did that you would want to upload the texel data to it and transition it to be the source image without losing data. If you're going to transition the image first to be a destination and then copy the texel data to it from a buffer, like we've done then you don't want to do this.

```c++
imageInfo.usage = VK_IMAGE_USAGE_TRANSFER_DST_BIT | VK_IMAGE_USAGE_SAMPLED_BIT;
```

This is just like a buffer usage. Here we are transfer to the image so it needs to be a destination, and setting the **VK_IMAGE_USAGE_SAMPLED_BIT** means we can put that in the shader.

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

Here you really should check if **VK_FORMAT_R8G8B8A8_SRGB** but it's so commonly supported that it's really not worth checking. You would have to do a lot of conversation to get to this work anyway.

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
##### Transitioning memory for images
We are actually going to use something similar to sychronisation with semaphores. The idea behind a **image memory barrier** is used to synchronised to ensure things like a write to a buffer is completed before reading from it. It also used to move image to a different queue which owns it, assuming that queue has **VK_SHARIN_MODE_EXCLUSIVE**.

Firstly I've abstracted the starting and ending of command buffer that are used for copying data. 
This is all covered in the [[Copying buffer data over]] as this is the first time it's used for anything practical.

```c++
static void transitionImageLayout(VkImage image, VkFormat format, VkImageLayout oldLayout, VkImageLayout newLayout) 
{
    VkCommandBuffer commandBuffer = beginOneTimeCommandBuffer(commandPool);

    VkImageMemoryBarrier barrier{};
    barrier.sType = VK_STRUCTURE_TYPE_IMAGE_MEMORY_BARRIER;
    barrier.oldLayout = oldLayout;
    barrier.newLayout = newLayout;

    endOneTimeCommandBuffer(commandBuffer, commandPool, mainQueue);
}
```

So here we begin by setting up the **VkImageMemoryBarrier** which will do the transition from on barrier to another. 

```c++
barrier.srcQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
barrier.dstQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
```

If you are using bariers that have transfer queue ownership you need to set these too fields up. You need to set them to **VK_QUEUE_FAMILY_IGNORED** if you don't care, these can't be 0.

```c++
barrier.image = image;
barrier.subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
barrier.subresourceRange.baseMipLevel = 0;
barrier.subresourceRange.levelCount = 1;
barrier.subresourceRange.baseArrayLayer = 0;
barrier.subresourceRange.layerCount = 1;
```

Then we set up the subresourceRange and the image. The subresourceRange is to do with the what part of the image we are interested same as the buffer we have done before. Since we aren't using mipmaps then **.levelCount** and **.layerCount** is equal to 1.

```c++
barrier.srcAccessMask = 0;
barrier.dstAccessMask = 0;
```

These barriers are for access masks before and after. This is exactly the same as the semaphores and what they wait on. So if you need to wait for part of the graphics pipeline to finish before you can do a transfer operation then you'll set things up these.

I'm gonna show the actual function call we will use to be able to talk about the arguments.
```c++
VKAPI_ATTR void VKAPI_CALL vkCmdPipelineBarrier(
    VkCommandBuffer                             commandBuffer,
    VkPipelineStageFlags                        srcStageMask,
    VkPipelineStageFlags                        dstStageMask,
    VkDependencyFlags                           dependencyFlags,
    uint32_t                                    memoryBarrierCount,
    const VkMemoryBarrier*                      pMemoryBarriers,
    uint32_t                                    bufferMemoryBarrierCount,
    const VkBufferMemoryBarrier*                pBufferMemoryBarriers,
    uint32_t                                    imageMemoryBarrierCount,
    const VkImageMemoryBarrier*                 pImageMemoryBarriers);
```

2) **srcStageMask** is everything that needs to happen before the barrier.
3) **dstStageMask** this is used to tell the pipeline what operations needs to be set before the pipeline is set.
For example, lets say you want to read a uniform after the barrier. You would need to specify the usage to be **VK_ACCESS_UNIFORM_READ_BIT** and the earliest shader part of the pipeline that reads that uniform **VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT**. Meaning that the **srcStageMask** is the thing we are waiting for which is the uniform read and because the fragment shader is going to read this uniform in this example we need to wait for the barrier to be completed before **VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT** operation can begin.

4)  **dependencyFlags** has 2 states **0** or **VK_DEPENDY_BY_REGION_BIT**. The region bit setting turns the barrier into a per-region condition. Meaning that when **VK_DEPENDY_BY_REGION_BIT** is set vulkan will start reading parts of the resource that have been written so far.

The rest deal with array's arrays of pipeline barriers types
5) **memoryBarrierCount** is the amount of **pMemoryBarriers** you have
6) **pMemoryBarriers** is the actual flat array of **VkMemoryBarrier** types
7) **bufferMemoryBarrierCount** is the amount of **pBufferMemoryBarriers** you have
8) **pBufferMemoryBarriers** is the actual flat array of **VkBufferMemoryBarrier**
9) **imageMemoryBarrierCount** is the amount of **pMemoryBarriers** you have
10) **pImageMemoryBarriers** is the actual flat array of **VkImageMemoryBarrier**

So lets build up the pipeline stages and the access flags
```c++
VkPipelineStageFlags sourceStage = VK_PIPELINE_STAGE_FLAG_BITS_MAX_ENUM;
VkPipelineStageFlags destinationStage = VK_PIPELINE_STAGE_FLAG_BITS_MAX_ENUM;

if (oldLayout == VK_IMAGE_LAYOUT_UNDEFINED && newLayout == VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL) 
{
    barrier.srcAccessMask = 0;
    barrier.dstAccessMask = VK_ACCESS_TRANSFER_WRITE_BIT;

    sourceStage = VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT;
    destinationStage = VK_PIPELINE_STAGE_TRANSFER_BIT;
}
else if (oldLayout == VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL && newLayout == VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL)
{
    barrier.srcAccessMask = VK_ACCESS_TRANSFER_WRITE_BIT;
    barrier.dstAccessMask = VK_ACCESS_SHADER_READ_BIT;

    sourceStage = VK_PIPELINE_STAGE_TRANSFER_BIT;
    destinationStage = VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT;
}
else 
{
    printf("Not support memory image layout transition");
}
    
vkCmdPipelineBarrier(commandBuffer, sourceStage, destinationStage, 0, 0, nullptr, 0, nullptr, 1, &barrier);
```

This code is meant to 2 state changes:
1) Transitioning an image access mask from undefined to be transfer writable and the stages we are going to wait for are **VK_PIPELINE_STAGE_TRANSFER_BIT**. All to do a copy from a buffer to an image.
2) Tranitioning an image access mask from transfer writable to shader read bit and the stages are going to wait for are **VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT** All to read the image in the fragment shader.

**VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT** and **VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT** are both fake parts of the pipeline but put us before everything begins and after everything has ended for our pipeline stage. It's also true that **VK_ACCESS_TRANSFER_WRITE_BIT** is a pseudo-stage like the other 2 but this makes sure we are at the point were things are transfered within the graphics pipeline.

You might actually wjat **VK_IMAGE_LAYOUT_GENERAL** for special cases here [[Introduction to texture mapping]]

When we run the **vkCmdPipelineBarrier** we are only interesting in **VkImageMemoryBarrier** so most of the rest is null. We just need to set the source and destination stages for our graphics pipeline to be waited on.

A quick note **vkCmdPipelineBarrier** has to be done on a graphics queue.
##### Copying a buffer to image
When we are doing this we are assuming the image to be copied is already in the optimal format.

First lets start with skeleton function before we begin
```c++
static void copyBufferToImage(VkBuffer buffer, VkImage image, uint32_t width, uint32_t height) 
{
    VkCommandBuffer commandBuffer = beginOneTimeCommandBuffer(transferCommandPool);

    endOneTimeCommandBuffer(commandBuffer, transferCommandPool, transferQueue)
}
```

First we set up the region copy we want to copy
```c++
VkBufferImageCopy region{};
region.bufferOffset = 0;
region.bufferRowLength = 0;
region.bufferImageHeight = 0;

region.imageSubresource.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
region.imageSubresource.mipLevel = 0;
region.imageSubresource.baseArrayLayer = 0;
region.imageSubresource.layerCount = 1;

region.imageOffset = { 0, 0, 0 };
region.imageExtent = { width, height, 1 };
```

**.bufferOffset** is how much we offset into the buffer to begin were the pixels start. **.bufferRowLength** and **.bufferImageHeight** are to do with how the pixels are layed out in memory. It's possible for example, to an image have padding between each row.

The rest of the code is to do with what you want to copy.

```c++
vkCmdCopyBufferToImage(commandBuffer, buffer, image, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL, 1, &region);
```

The second and third arguments are part of the arguments. Again we are assuming the image to be copied is already in the optimal format hence the **VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL**.. Then you select the region you want to copy, like normal you can copy buffer to images in arrays.
##### Preparing the texture image
I'm gonna show the complete **createTextureImage** function to dementrate what needs to happen from beginning to end. I've written this notes in comments because it give maximal context to what's going on.
```c++
static void createTextureImage(const char* path, VkImage& textureImage, VkDeviceMemory& textureImageMemory)
{
    int texWidth;
    int texHeight;
    int channles;

	//Here load the raw pixel data to an unsigned char* 
    stbi_uc* pixels = stbi_load(path, &texWidth, &texHeight, &channles, STBI_rgb_alpha);
    //We also need the device size for the staging buffers.
    VkDeviceSize imageSize = texWidth * texHeight * 4;
	//Next we check if it's loaded correctly
    if (!pixels)
    {
        printf("Failed to load texture image.");
    }
	//Then we start with the staging buffers.
    VkBuffer imageStagingBuffer;
    VkDeviceMemory imageStagingBufferMemory;

	//We are making the staging buffer for copying to an image so we use VK_BUFFER_USAGE_TRANSFER_SRC_BIT to transfer and it needs to he host visible so we can load data in.
    createBuffer(imageSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, imageStagingBuffer, imageStagingBufferMemory);

	//Next we copy the data to the staging buffer using the imageSize we calculated eariler when we loaded the image from disk.
    void* imageData;
    vkMapMemory(device, imageStagingBufferMemory, 0, imageSize, 0, &imageData);
    memcpy(imageData, pixels, size_t(imageSize));
    vkUnmapMemory(device, imageStagingBufferMemory);

	//After that we free the pixels as they are in the staging buffer.
    stbi_image_free(pixels);

	//Next we create the image we are going to copy the buffer into. We aren't doing anything special but it needs to be created with VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT as it will live on the GPU and be used to sampled on VK_IMAGE_USAGE_SAMPLED_BIT and it needs to be the destination for our copy so we need the VK_IMAGE_USAGE_TRANSFER_DST_BIT to be ORed with it..
    createImage(texWidth, texHeight, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_TILING_OPTIMAL, VK_IMAGE_USAGE_TRANSFER_DST_BIT | VK_IMAGE_USAGE_SAMPLED_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, textureImage, textureImageMemory);

	//The we transition image ready to receive data.  So the image needs to be have the same format and type as the usage type.
    transitionImageLayout(textureImage, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_LAYOUT_UNDEFINED, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL);
    //Next we copy the entire image from the buffer to the image.
    copyBufferToImage(imageStagingBuffer, textureImage, uint32_t(texWidth), uint32_t(texHeight));
    //After that we transition the image memory layout to be read by the shader. 
    transitionImageLayout(textureImage, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL, VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL);

	//Finally we can destroy the staging buffers.
    vkDestroyBuffer(device, imageStagingBuffer, nullptr);
    vkFreeMemory(device, imageStagingBufferMemory, nullptr);
}
```

Note this code be improved a lot. We are creating command buffers and waiting for the queue to go idle before we set up another one and repeat. You can flush and set up command buffers in away that everythings in 1 queue and submitted together.

You'll also want to clean up the VkImage and VkDeviceMemory

```c++
VkImage image;
VkDeviceMemory imageMemory;
createTextureImage("Textures/frobert.png", image, imageMemory);

//A load of stuff here...

vkDestroyImage(device, image, nullptr);
vkFreeMemory(device, imageMemory, nullptr);
```
##### Texture image view
Right if you have completed the steps within [[Preparing the texture image]] you'll need 2 extra things surround the VkImage, the VkSampler and the VkImageView.

The VkImageView which contains all the metadata about the image this is almost exactly the same as [[Creating an image view]] apart from the fact we are using a VkImage from [[Preparing the texture image]] instead of the swapchain. So look at that for more details.

Make sure you clean up the image view.
##### Samplers
You can send up an image to the shaders read directly from it, however that's never really done with textures. Typically you'll want sampler that applies filters and transforms that computes the final image.

An oversampling might is when have some geometry that has more fragments than texels for a given texture. If you apply filtering you get the first one. With a sampler though you can linearly interpolate between the 4 closest texels you get the second picture.

![[texture_filtering.png]]

Undersampling is the opposite problem were you have too many texels for fragments and you get weird artifacts when you have sharp contrasting textures. You can apply anisotropic filtering to help with this.

![[anisotropic_filtering.png]]

Samplers also details with what happens when you sample outside the texture on geometry, in the samplers **address mode**. Here are pictures of the possible abilities

![[texture_addressing.png]]
##### Creating a sampler
We are going to use this function to create a texture.

```c++
static VkSampler createSampler() { /*stuff here*/ }
```

We begin by making the sampler creation struct.
```c++
VkSamplerCreateInfo samplerInfo{};
samplerInfo.sType = VK_STRUCTURE_TYPE_SAMPLER_CREATE_INFO;
samplerInfo.magFilter = VK_FILTER_LINEAR;
samplerInfo.minFilter = VK_FILTER_LINEAR;
```

The **.magFilter** and **.minFilter** states how the texels will be interpolated with either magnification with is concerned with oversampling and minified for undersampling. 
- **VK_FILTER_LINEAR** is for linear interpolation
- **VK_FILTER_NEAREST** is for no interpolation.

```c++
samplerInfo.addressModeU = VK_SAMPLER_ADDRESS_MODE_REPEAT;
samplerInfo.addressModeV = VK_SAMPLER_ADDRESS_MODE_REPEAT;
samplerInfo.addressModeW = VK_SAMPLER_ADDRESS_MODE_REPEAT;
```

These are to do with how the texels are handled if you go outside of the texture image. As we are texture space the coordinate axes are U, V, W instead of X, Y, Z. Meaning you can have a different sampler modes for different axes.

You have these 5 to choose from.
- **VK_SAMPLER_ADDRESS_MODE_REPEAT**
- **VK_SAMPLER_ADDRESS_MODE_MIRRORED_REPEAT**
- **VK_SAMPLER_ADDRESS_MODE_CLAMP_TO_EDGE** 
- **VK_SAMPLER_ADDRESS_MODE_CLAMP_TO_BORDER**
- **VK_SAMPLER_ADDRESS_MODE_MIRROR_CLAMP_TO_EDGE**

For more information on what this looks like look at [[What are samplers]] note.

```c++
samplerInfo.anisotropyEnable = VK_TRUE;
samplerInfo.maxAnisotropy = maxAnisotropy;
```

When **.anisotropyEnable** is true this allows use to increase the amount of texel samples that can be used for calculating the final image. This will increase the quality of the image but reduce performance. The overhead seems to be rather low so it's a recommended features to use.

However we also need to know what the **.maxAnisotropy** and to turn on that feature on the graphics card before we can use this. **maxAniostropy** is a uint32_t

First thing we need to do when selecting an appropriate GPU is get the max aniostropy from our selected GPU
```c++
VkPhysicalDeviceProperties properties;
vkGetPhysicalDeviceProperties(gpus[i], &properties);

//... score being checked here for the best GPU

if (highestScore < score) 
{
    highestScore = score;
    bestGPUIndex = i;
    maxAnisotropy = properties.limits.maxSamplerAnisotropy;
}
```

After that we will need to enable this anisotropy feature on the GPU with the **VkPhysicalDeviceFeatures2** struct.

```c++
deviceFeatures.features.samplerAnisotropy = VK_TRUE;
```

Then everything should. However not all graphics will support this so you'll need to check in the but for now, I will again assume that most people running this application will have a graphics card made in the last 10 years

```c++
samplerInfo.anisotropyEnable = VK_FALSE;
samplerInfo.maxAnisotropy = 1.f;
```

The code above will turn off anisotropy.

```c++
samplerInfo.borderColor = VK_BORDER_COLOR_INT_OPAQUE_BLACK;
```

This tells the sampler if the add a border when sampling beyond when **VK_SAMPLER_ADDRESS_MODE_CLAMP_TO_BORDER** address mode is used. You can do black, white, transparent or a custom colour with a vulkan extension.

```c++
samplerInfo.unnormalizedCoordinates = VK_FALSE;
```

If this is set to true, you can sample from `[0, textureWidth)` and `[0, textureHeight)` you almost never want this because you'll often be working with textures of different sizes so normalised to `[0, 1)` and `[0, 1)` is best.

```c++
samplerInfo.mipmapMode = VK_SAMPLER_MIPMAP_MODE_LINEAR;
samplerInfo.mipLodBias = 0.f;
samplerInfo.minLod = 0.f;
samplerInfo.maxLod = 0.f;
```

These are all to do with mipmapping if you're not mipmapping this is the set up you'll want to have.

```c++
VkSampler sampler;
if (vkCreateSampler(device, &samplerInfo, nullptr, &sampler) != VK_SUCCESS) 
{
    printf("Could not create sampler");
}
```

The sampler can be applied to any VkImage you want. That's why the VkImage and VkSampler are different objects.

Don't forget to destroy the sampler.

```c++
vkDestroySampler(device, textureSampler, nullptr);
```
##### Combined image sampler
To allow us to access an image view that contains a texture in a shader we need to be able to use a descriptor set type called a **combined image sampler**. 

The first thing you have to do is create a sampler descriptor with **VkDescriptorSetLayoutBinding** and add it to the descriptor then add that to the **DescriptorSetLayoutCreateInfo** This is the same as [[Creating a descriptor set layout]] but we make it a combined image sampler.

```c++
VkDescriptorSetLayoutBinding sampleLayoutBinding{};
sampleLayoutBinding.binding = 1;
sampleLayoutBinding.descriptorCount = 1;
sampleLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
sampleLayoutBinding.pImmutableSamplers = nullptr;
sampleLayoutBinding.stageFlags = VK_SHADER_STAGE_FRAGMENT_BIT;
    
//Pipeline layout creation
std::array<VkDescriptorSetLayoutBinding, 2> binding = { uboLayoutBinding, sampleLayoutBinding };
VkDescriptorSetLayoutCreateInfo layoutInfo{};
layoutInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_LAYOUT_CREATE_INFO;
layoutInfo.bindingCount = uint32_t(binding.size());
layoutInfo.pBindings = binding.data();
```

Here we are creating the descriptor but this time it's a **VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER** type. You can put image sampler in the vertex shader if you need to generate a height map, by deforming a vertex grid.

We also need to increase the size of the descriptor pool to include the sampler
```c++
//Descriptor pool creation
std::array<VkDescriptorPoolSize, 2> poolSize{};
poolSize[0].type = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
poolSize[0].descriptorCount = imageCount;
poolSize[1].type = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
poolSize[1].descriptorCount = imageCount;

VkDescriptorPoolCreateInfo poolDescriptorCreateInfo{};
poolDescriptorCreateInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_POOL_CREATE_INFO;
poolDescriptorCreateInfo.poolSizeCount = uint32_t(poolSize.size());
poolDescriptorCreateInfo.pPoolSizes = poolSize.data();
poolDescriptorCreateInfo.maxSets = imageCount;
```

Since the allocation of the descriptors and left to the graphics driver you don't need to set things up in away that types are exact, but it's best practice to do so.

Next we'll need to write the the image sampler descriptor set to be able to bind this at draw time.
```c++
VkDescriptorBufferInfo desBufferInfo{};
desBufferInfo.buffer = uniformBuffers[i];
desBufferInfo.offset = 0;
desBufferInfo.range = sizeof(ModelData);

VkDescriptorImageInfo desImageInfo{};
desImageInfo.imageLayout = VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL;
desImageInfo.imageView = textureImageView;
desImageInfo.sampler = textureSampler;

std::array<VkWriteDescriptorSet, 2> descriptorWrites{};

descriptorWrites[0].sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
descriptorWrites[0].dstSet = descriptorSets[i];
descriptorWrites[0].dstBinding = 0;
descriptorWrites[0].dstArrayElement = 0;
descriptorWrites[0].descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
descriptorWrites[0].descriptorCount = 1;
descriptorWrites[0].pBufferInfo = &desBufferInfo;
descriptorWrites[0].pImageInfo = nullptr;
descriptorWrites[0].pTexelBufferView = nullptr;

descriptorWrites[1].sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
descriptorWrites[1].dstSet = descriptorSets[i];
descriptorWrites[1].dstBinding = 1;
descriptorWrites[1].dstArrayElement = 0;
descriptorWrites[1].descriptorType = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
descriptorWrites[1].descriptorCount = 1;
descriptorWrites[1].pBufferInfo = nullptr;
descriptorWrites[1].pImageInfo = &desImageInfo;
descriptorWrites[1].pTexelBufferView = nullptr;

vkUpdateDescriptorSets(device, uint32_t(descriptorWrites.size()), descriptorWrites.data(), 0, nullptr);
```

Note we are updating the **.pImageInfo** with out image data with unlike the **VkDescriptorBufferInfo** struct.
##### Texture coordinates
You need to have vertex attribute descriptions for the vertex buffer which is were the texcoords are gonna live. 
```c++
struct Vertex 
{
    vec2s position;
    vec3s colour;
    vec2s texCoord;

    static std::array<VkVertexInputAttributeDescription, 3> getAttributeDescriptions() 
    {
        std::array<VkVertexInputAttributeDescription, 3> attributeDescriptions{};

        attributeDescriptions[0].binding = 0;
        attributeDescriptions[0].location = 0;
        attributeDescriptions[0].format = VK_FORMAT_R32G32_SFLOAT;
        attributeDescriptions[0].offset = offsetof(Vertex, position);

        attributeDescriptions[1].binding = 0;
        attributeDescriptions[1].location = 1;
        attributeDescriptions[1].format = VK_FORMAT_R32G32B32_SFLOAT;
        attributeDescriptions[1].offset = offsetof(Vertex, colour);

        attributeDescriptions[2].binding = 0;
        attributeDescriptions[2].location = 2;
        attributeDescriptions[2].format = VK_FORMAT_R32G32_SFLOAT;
        attributeDescriptions[2].offset = offsetof(Vertex, texCoord);

        return attributeDescriptions;
    }    
```

Note the format being **VK_FORMAT_R32G32_SFLOAT** matching the `vec2s`. 

You also have to have the data within your vertex buffer
```c++
const std::vector<Vertex> vertices = 
{
    {{ -0.5f, -0.5f }, { 1.f, 1.f, 1.f }, { 1.f, 0.f }},
    {{  0.5f, -0.5f }, { 1.f, 1.f, 1.f }, { 0.f, 0.f }},
    {{  0.5f,  0.5f }, { 1.f, 1.f, 1.f }, { 0.f, 1.f }},
    {{ -0.5f,  0.5f }, { 1.f, 1.f, 1.f }, { 1.f, 1.f }}
};
```

This just texture maps a sqaure. It's not that important what the data because it's just dummy data it's more that it matches with the **VK_FORMAT_R32G32_SFLOAT**. 
##### Using textures with shaders
The first thing you need to do is be aware the vertex attribute that comes through the vertex buffer needs to handle. So you need to pass the texture coordinates from the vertex shader to the fragment shader.

```c
#version 450
layout (location = 0) in vec2 inPosition;
layout (location = 1) in vec3 inColour;
layout (location = 2) in vec2 inTexCoord; 

layout (location = 0) out vec3 fragColour;
layout (location = 1) out vec2 fragTexCoord;

void main()
{
	gl_Position = modelData.proj * modelData.view * modelData.model * vec4(inPosition, 0.0, 1.0);
	fragColour = inColour;
	fragTexCoord = inTexCoord;
}
```

Here we are just going to print the texture coordinate with the texture coordinates we passed over from the vertex shader.
```c
layout(location = 0) in vec3 fragColour;
layout(location = 1) in vec2 fragTexCoord;

layout(location = 0) out vec4 outColour;

void main()
{
	outColour = vec4(fragTexCoord, 0.0, 1.0);
}
```

If you're having a problem with textures on models you can actually just display the texture coordinates directly. This can be thought of as shader "printf" debugging as it shows you if the texture coordinates make sense.

![[texture_coordinates.png]]
This is what a correct texture coordinates should look like. The green represents the hoziontal coordinates and the red channel represents the coordinates. The yellow and the black corners show that we are interperlating correctly from 0, 0 to 1, 1 across the square.

You can see that if I make a mistake with the texture coordinates I don't get this picture.

![[texture_coordinate_wrong.png]] 

Since we are missing the black corner you can get an idea on what's wrong here.

To actually use the combined image sampler you'll want to create a uniform **sampler** object in the shader you want to use a texture.

```c
layout(binding = 1) uniform sampler2D texSampler;
```

You have 1D and 3D version of this so be aware on what type of texture you're interested in. The binding has to match the `descriptorWrites[1].dstBinding = 1;` when we created the `std::array<VkWriteDescriptorSet, 2>` when binding the image buffer.

```c
vec4 tex = texture(texSampler, fragTexCoord);
outColour = vec4(fragColour * tex.rgb, tex.a);
```

Next you'll want to use the **texture** function in your shader that deals with the sampling transformation and filtering in the background. 

I only added a little extra code here so I could add the **fragColour** to the RBG parts of the texture while leaving the alpha pixels alone.

```c
vec4 tex = texture(texSampler, fragTexCoord * 4.0);
outColour = vec4(fragColour * tex.rgb, tex.a);
```

You can if you want mess around the with texture coordinate values and you'll see the address mode at work for a given sampler. 

This is the output of that result
![[texture_frogx4.png]]