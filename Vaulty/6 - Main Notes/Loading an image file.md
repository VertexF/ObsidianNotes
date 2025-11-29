2025-11-27 11:58
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]]
# Loading an image file

I'll be using the fantastic library for loading images the stb_image.h. So include that in your project and forget to add the **STB_IMAGE_IMPLEMENTATION** declaration before the include.
```c++
#define STB_IMAGE_IMPLEMENTATION
#include <stb_image.h>
```

I will be brief because I've done this before. This is how you load an image with stbi_image

```c++
//"Textures/frobert.png"
static void createTextureImage(const char* path)
{
    int texWidth;
    int texHeight;
    int channles;

    stbi_uc* pixels = stbi_load(path, &texWidth, &texHeight, &channles, STBI_rgb_alpha);
    VkDeviceSize bufferSize = texWidth * texHeight * 4;

    if (!pixels)
    {
        printf("Failed to load texture image.");
    }
}
```

We have here the total memory size of the image with **VkDeviceSize** and the raw pixels.

```c++
VkBuffer imageStagingBuffer;
VkDeviceMemory imageStagingBufferMemory;

createBuffer(imageSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, imageStagingBuffer, imageStagingBufferMemory);

void* imageData;
vkMapMemory(device, imageStagingBufferMemory, 0, imageSize, 0, &imageData);
memcpy(imageData, pixels, size_t(imageSize));
vkUnmapMemory(device, imageStagingBufferMemory);

stbi_image_free(pixels);
```

The creation of the staging buffer is exactly the same as [[Creating the vertex buffer with staging buffers]]

Don't forget to free the pixels once the memcpy has happend.
# References
##### Main Notes
[[Introduction to texture mapping]]
[[Creating the vertex buffer with staging buffers]]
#### Source Notes
[[Texture mapping]]