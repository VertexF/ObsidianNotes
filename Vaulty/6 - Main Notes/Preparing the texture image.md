2025-11-27 19:27
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]]
# Preparing the texture image

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

	//Next we create the image we are going to copy the buffer into. We aren't doing anything special but it needs to be created with VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT as it will live on the GPU and be used to sampled on VK_IMAGE_USAGE_SAMPLED_BIT and it needs to be the destination for our copy so we need the VK_IMAGE_USAGE_TRANSFER_DST_BIT to be OR-ed with it..
    createImage(texWidth, texHeight, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_TILING_OPTIMAL, VK_IMAGE_USAGE_TRANSFER_DST_BIT | VK_IMAGE_USAGE_SAMPLED_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, textureImage, textureImageMemory);

	//The we transition image ready to receive data. So the image needs to be have the same format and type as the usage type.
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
# References
##### Main Notes
[[Introduction to texture mapping]]
[[Loading an image file]]
[[Creating a VkImage for texture mapping]]
[[Transitioning image memory for texture mapping]]
#### Source Notes
[[Texture mapping]]