2025-10-17 19:47
Status: #baby 
Tags: [[vulkan]] [[vulkan swapchain]]
# Getting swapchain images

If you've created the swapchain you also need to create the VkImage's that you're going to use with the swapchain. These get destroyed with the swapchain so you don't need to worry about destroying them.

You just assume the number of VkImage's will be 4 or 3 or whatever. We did this in the note [[Querying surface capabilities]] at the end of the note we had with the image acount. 

We follow the very standard querying pattern with this to get a container of images we are going to use. 

```c++
vkGetSwapchainImagesKHR(device, swapchain, &imageCount, nullptr);
std::vector<VkImage> swapchainImages(imageCount);
vkGetSwapchainImagesKHR(device, swapchain, &imageCount, swapchainImages.data());
```

If you don't have access to the **VkFormat** from the **VkSurfaceFormatKHR** struct or the **VkExtent2D** from the **VkSurfaceCapabilitiesKHR** struct you'll need both values stored for later.
# References
##### Main Notes
[[Creating the swapchain]]
[[Querying surface capabilities]]
#### Source Notes
[[Vulkan-Tutorial]]