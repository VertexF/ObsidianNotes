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

I believe to make SDL3 DPI aware you need to set the **SDL_WINDOW_HIGH_PIXEL_DENSITY** which will try to create a window that has the correct DPI. This is from the documentation 
"`If the window is created with the `SDL_WINDOW_HIGH_PIXEL_DENSITY` flag, SDL will try to match the native pixel density for the display, otherwise it will try to have the pixel size match the window size.`" - https://wiki.libsdl.org/SDL3/README-highdpi

Finally we need to get the minimum number of images that the hardware supports for the swapchain. You can't just assume it's 3 or 2 you have to check. 

```c++
uint32_t imageCount = surfaceCapabilities.minImageCount + 1;

if (surfaceCapabilities.maxImageCount > 0 && imageCount > surfaceCapabilities.maxImageCount)
{
	imageCount = surfaceCapabilities.maxImageCount;
}
```

If the value if the maxImageCount is **GREATER** than 0 that means there **is** a maximum value. Meaning we need to make sure our value of **.minImageCount + 1** isn't too large with extra check.
# References
##### Main Notes
[[Checking surface and swapchain compability]]
#### Source Notes
[[Drawing a triangle]]