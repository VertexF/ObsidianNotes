2025-10-17 13:01Status: #adult  
Tags: [[vulkan]] [[vulkan surface]] [[vulkan swapchain]]
# Querying surface capabilities

The first thing you need to do is get the surface capabilities which is simple enough.

```c++
VkSurfaceCapabilitiesKHR surfaceCapabilities;
vkGetPhysicalDeviceSurfaceCapabilitiesKHR(physicalDevice, surface, &surfaceCapabilities);
```

Then you'll want to get the swapchain extents from the struct, with:

```c++
VkExtent2D swapchainExtent = surfaceCapabilities.currentExtent;
if (swapchainExtent.width == UINT32_MAX)
{
	swapchainExtent.width = clamp(swapchainExtent.width, surfaceCapabilities.minImageExtent.width, surfaceCapabilities.maxImageExtent.width);
	swapchainExtent.height = clamp(swapchainExtent.height, surfaceCapabilities.minImageExtent.height, surfaceCapabilities.maxImageExtent.height);
}
```

Here you might notice that we clamping the values between the minimum and the maximum values from the **VkSurfaceCapabilitiesKHR** this is to make sure the pixel width and height doesn't fall outside the range **IF** the width is reading the value to be the max UINT32_MAX value.

In SDL3 there is no function call to handle high DPI display's at the moment meaning that if vulkan hasn't updated to detect this the pixels width and height will be more than the actual resolution of the application window then this doesn't work for high DPI. This needs testing.

Finally we need to get the minimum number of images that the hardware supports for the swapchain. You can't just assume it's 3 or 2 you have to check. 

```c++
uint32_t imageCount = surfaceCapabilities.minImageCount + 1;

if (surfaceCapabilities.maxImageCount > 0 && imageCount > surfaceCapabilities.maxImageCount)
{
	imageCount = surfaceCapabilities.maxImageCount;
}
```

If the value if the maxImageCount is **GREATER** than 0 that means there is a maximum value. Meaning we need to make sure our value of **.minImageCount + 1** isn't too large with extra check.
# References
##### Main Notes
[[Checking surface and swapchain compability]]
#### Source Notes
[[Drawing a triangle]]