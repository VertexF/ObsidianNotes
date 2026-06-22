2026-06-08 11:52
Status: #baby 
Tags: [[vulkan]] [[graphics theory]] [[physically based rendering]]
# Swapchain colour space format

When you are dealing HDR/LDR images and doing some kind of lighthing in a fragment like PBR. You have to be careful that the swapchain image format isn't already coverting your image to linear space, as you can do this twice by accident making everything look grey.

At a very basic level if you're going to covert the colour space yourself you need to set the swapchain image format to be a `UNORM` version of colour space. If you're not doing the coversion you need to set this `SRBG`.

```c++
VkSwapchainCreateInfoKHR swapchainCreateInfo{};
swapchainCreateInfo.sType = VK_STRUCTURE_TYPE_SWAPCHAIN_CREATE_INFO_KHR;
swapchainCreateInfo.imageFormat = 
VK_FORMAT_B8G8R8A8_UNORM; //For non-linear space
VK_FORMAT_B8G8R8A8_SRGB; //For linear space conversion

vkCreateSwapchainKHR(vulkanDevice, &swapchainCreateInfo, nullptr, &vulkanSwapchain);
```

This colour imageFormat is how the image will be interpreted by the presentation engine. The conversion happens while doing a read/write operation.
# References
##### Main Notes
[[What is linear space]]
[[The problem with LDR cubemaps in PBR]]
#### Source Notes
[[Diffuse irradiance]]
