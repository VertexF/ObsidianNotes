2025-10-17 13:17
Status: #baby 
Tags: [[vulkan]] [[vulkan surface]] [[vulkan swapchain]]
# Checking surface format

Here we need to check if the type of pixel data we want to write to our VkImages's is supported or not. You query these is a very standard vulkan way with 

```c++
uint32_t formatCount = 0;
vkGetPhysicalDeviceSurfaceFormatsKHR(physicalDevice, surface, &formatCount, nullptr);
std::vector<VkSurfaceFormatKHR> surfaceFormats(formatCount);
if (formatCount != 0)
{
	vkGetPhysicalDeviceSurfaceFormatsKHR(physicalDevice, surface, &formatCount, surfaceFormats.data());
}
```

Then you'll want to loop through the array of **VkSurfaceFormatKHR** and select the that has the colour pixel format you want. The most widely supported on is **VK_FORMAT_B8G8R8A8_SRGB** which is a good default to select for.

```c++
	VkSurfaceFormatKHR swapchainSurface{};
	for (const auto& availableSurface : surfaceFormats)
	{
		if (availableSurface.format == VK_FORMAT_B8G8R8A8_SRGB) 
		{
			swapchainSurface = availableSurface;
		}
	}
	swapchainSurface = surfaceFormats[0];
```

Notice the last time of this code snippet. It's totally fine here to select the first format in the list of you can't find the one you want.
# References
##### Main Notes
[[Checking surface and swapchain compability]]
#### Source Notes
[[Vulkan-Tutorial]]