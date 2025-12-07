2025-12-06 19:28
Status: #baby 
Tags: [[vulkan]] [[depth testing]]
# Resizing the window with a depth buffer

You need to recreate the depth buffer on a window resize because the depth buffer resolution should change with the windows

```c++
static void recreateSwapchain()
{
    vkDeviceWaitIdle(device);

    vkGetPhysicalDeviceSurfaceCapabilitiesKHR(physicalDevice, surface, &surfaceCapabilities);
    swapchainExtent = surfaceCapabilities.currentExtent;

    if(swapchainExtent.width == 0 || swapchainExtent.height == 0) 
    {
        return;
    }

    cleanupSwapchain();

    createSwapchain();
    createImageViews();
    createDepthResources();
    createFramebuffers();
}
```

The function **createDepthResources** contains the code for the [[Creating the depth image and view]] 
# References
##### Main Notes
[[Creating the depth image and view]]
#### Source Notes
[[Depth buffering]]