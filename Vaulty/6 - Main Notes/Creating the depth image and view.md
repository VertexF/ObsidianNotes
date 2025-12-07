2025-12-06 17:04
Status: #baby 
Tags: [[vulkan]] [[depth testing]]
# Creating the depth image and view

We need to now create a depth attachment which is similar to when we were [[Creating a VkImage for texture mapping]] but this time we are going to make image depth instead of image colour. We need to create a depth attachment that's going to live for the entire life time of the swapchain. We can't use a simply do what we did with [[Getting swapchain images]] we will have to make our own.

Since this depth attachment is going to be part of the swapchain it needs to be re-created with the swapchain. So I'm going to wrap everything in function to make re-creation of the swapchain easier.

When creating the depth image the same resolutation as the colour attachment which will come from the swapchain extents, optimal tiling and device local memory. We also need to work out the special depth format we want to use. Again this is all similar to the [[Creating a VkImage for texture mapping]] 

Unlike with texture image we don't need to be specific to how we access the VkImage. If you remember the a texture image needs a VK_FORMAT_R8G8B8A8_SRGB to be able to be accessed in the fragment shader, but because we aren't going to be accessing the values directly in the fragment shader we have some flexibility. The formats you would want are:
- **VK_FORMAT_D32_SFLOAT** for 32-bit float for depth.
- **VK_FORMAT_F32_SFLOAT_8_UINT** for 32-bit signed flaot for depth and 8 bit stencil component
- **VK_FORMAT_D24_UNORM_S8_UINT** for 24-bit float for depth and 8 bit stencil component

Like with colour formats not all these depth formats are supported. So lets begin by writing a helper function to work out what's support or not.

```c++
static VkFormat findSupportedFormat(const std::vector<VkFormat>& candidtes, VkImageTiling tiling, VkFormatFeatureFlags features) 
{
    for (VkFormat format : candidtes) 
    {
        VkFormatProperties properties;
        vkGetPhysicalDeviceFormatProperties(physicalDevice, format, &properties);

        if (tiling == VK_IMAGE_TILING_LINEAR && (properties.linearTilingFeatures & features) == features) 
        {
            return format;
        }
        else if (tiling == VK_IMAGE_TILING_OPTIMAL && (properties.optimalTilingFeatures & features) == features)
        {
            return format;
        }
    }

    assert(!"No supported format based on the feature enum %d", features);
}
```

At a basic level this function loops through the vector **candidtes** and check to see if it support the either **VK_IMAGE_TILING_LINEAR** or **VK_IMAGE_TILING_OPTIMAL** with the `VkFormatProperties properties;` It's possible for the condition to be non-zero when doing the **AND** operation so we do an `==` operation to make sure that is the thing we want.

What are these features though? Well first you need to know what the tiling is this is discussed in [[Creating a VkImage for texture mapping]] but basicly linear is for images you want in major-row order to change later and optimial is implementation defined order.
- **properties.linearTilingFeatures** this tells us that format supports linear tiling.
- **properties.optimalTilingFeatures** this tell us that format supports optimal tiling
- **properties.bufferFeatures** Uses cases that are supported for buffers.

```c++
static VkFormat findDepthFormat() 
{
    return findSupportedFormat({VK_FORMAT_D32_SFLOAT, VK_FORMAT_D32_SFLOAT_S8_UINT, VK_FORMAT_D24_UNORM_S8_UINT},
        VK_IMAGE_TILING_OPTIMAL, VK_FORMAT_FEATURE_DEPTH_STENCIL_ATTACHMENT_BIT);
}
```

In this function we are taking the **findSupportedFormat** vector and ordering the formats we want the most to least and checking if any of them support **VK_FORMAT_FEATURE_DEPTH_STENCIL_ATTACHMENT_BIT** when we are **VK_IMAGE_TILING_OPTIMAL**.

Here we are very likely to skip over the stencil part of the depth buffer, I'm setting up the simplest depth buffer but please re-order things to your needs.

```c++
VkFormat depthFormat = findDepthFormat();

createImage(swapchainExtent.width, swapchainExtent.height, depthFormat, VK_IMAGE_TILING_OPTIMAL, VK_IMAGE_USAGE_DEPTH_STENCIL_ATTACHMENT_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, depthImage, depthImageMemory);
createImageView(depthImage, depthFormat, VK_IMAGE_ASPECT_DEPTH_BIT);
```

Now we create the depth image with the format, usage and memory properties. After that we can create the image view on top of the image that handles depth. 

This image is special we don't need to do any map memory operations because this will be cleared at the start of the render pass.
# References
##### Main Notes
[[3D geometry without the depth buffer]]
[[Creating a VkImage for texture mapping]]
[[Getting swapchain images]]
#### Source Notes
[[Depth buffering]]