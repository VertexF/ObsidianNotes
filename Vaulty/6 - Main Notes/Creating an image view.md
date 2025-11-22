2025-10-19 12:10
Status: #baby 
Tags: [[vulkan]] [[vulkan image views]] [[vulkan swapchain]]
# Creating an image view

This is going to explain how to create a image view from a swapchain image. This is the simplest VkImageView you can make. This is for if you're simply writing the graphics straight to a VkImage.

So if you're going to create a VkImageView you'll need to use the **VkImageViewCreateInfo** struct. The VkImage we are going to use is a swapchain image. 

```c++
VkImageView swapchainImageView;

VkImageViewCreateInfo createImageViewInfo{};
createImageViewInfo.sType = VK_STRUCTURE_TYPE_IMAGE_VIEW_CREATE_INFO;
createImageViewInfo.image = swapchainImages;
```

Next you'll want to specify how the data should be interpreted with **.viewType** and **.format**. The types **VK_IMAGE_VIEW_TYPE_1D|2D|3D|CUBE** are used in different contexts for types of data. 2D is standard for regular textures and for a swapchain image.

The format depends on what you're dealing with, if you're dealing with swapchain image you can just take the swapchain format from the **VkSurfaceFormatKHR** with **.format**

```c++
	createImageViewInfo.viewType = VK_IMAGE_VIEW_TYPE_2D;
	createImageViewInfo.format = swapchain.format;
```

Next you'll want to set up the colour components swizzle values. What this allows is rearragement of colour channels or set the from a range from 1 to 0. Changing these values can result in monochromic images or other effects. For swapchain images we just want the defaults.

```c++
	createImageViewInfo.components.r = VK_COMPONENT_SWIZZLE_IDENTITY;
	createImageViewInfo.components.g = VK_COMPONENT_SWIZZLE_IDENTITY;
	createImageViewInfo.components.b = VK_COMPONENT_SWIZZLE_IDENTITY;
	createImageViewInfo.components.a = VK_COMPONENT_SWIZZLE_IDENTITY;
```

Finally you'll want to set up the **.subresourceRange** values which explain the purpose of the VkImage and now much we want to access. This is were the mip levels are set for example. For swapchain image we to say it's a colour image and we need the whole thing with no mips.

```c++
	createImageViewInfo.subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
	createImageViewInfo.subresourceRange.baseMipLevel = 0;
	createImageViewInfo.subresourceRange.levelCount = 1;
	createImageViewInfo.subresourceRange.baseArrayLayer = 0;
	createImageViewInfo.subresourceRange.layerCount = 1;
```

Just a word of warning about **.levelCount** and **.layerCount** they can't be 0 and it's a very common mistakenly type out layerCount or levelCount twice and leave the other one as 0. So be careful.

Then you just want to create the VkImageView with the creation function.

```c++
if (vkCreateImageView(device, &createImageViewInfo, nullptr, &swapchainImageView) != VK_SUCCESS)
{
	printf("Failed to create image view");
}
```

This can fail so you need to check. 
# References
##### Main Notes
[[What are image views]]
[[What is a swapchain]]
[[Destroying a image view]]
#### Source Notes
[[Drawing a triangle]]