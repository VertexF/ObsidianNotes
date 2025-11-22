2025-09-23 07:42
Status: #baby 
Tags: [[vulkan]] [[queue families]]
# Checking for family queue suitability

Now if you've selected a good GPU you'll need to select the correct queue family. If you want to render you'll need at least to use the **VK_QUEUE_GRAPHICS_BIT**

The downside of just using a graphics queue for everything is that it can be a little slower if a draw command takes a long time to submit, then the transfer command has to wait. It's normally good enough for most things. With the exception of the presentation queue, you'll want the graphics queue and the presentation queue to be the same for speed.

What you want to is get the index of the queue family you are interested which is a uint32_t. For example, we want the index of the our main queue family index to be the one that contain both graphics and compute capability, so we can set up the VkQueue from the main queue family. 
```c++
uint32_t mainQueueFamilyIndex = UINT32_MAX;
uint32_t computeQueueFamilyIndex = UINT32_MAX;
uint32_t computeQueueIndex = UINT32_MAX;
for (uint32_t familyIndex = 0; familyIndex < queueFamilyCount; ++familyIndex)
{
	//Get the queue family we are working on.
	VkQueueFamilyProperties queueFamily = queueFamilies[familyIndex];
	//Reject that queue family if he has nothing in it.
    if (queueFamily.queueCount == 0)
    {
	    continue;
    }
    
    //Main queue is the one with both graphics and compute
    if ((queueFamily.queueFlags & (VK_QUEUE_GRAPHICS_BIT | VK_QUEUE_COMPUTE_BIT)) == (VK_QUEUE_GRAPHICS_BIT | VK_QUEUE_COMPUTE_BIT))
    {
	    //The index of main and compute are the family next.
        mainQueueFamilyIndex = familyIndex;

        if (queueFamily.queueCount > 1)
        {
            computeQueueFamilyIndex = familyIndex;
            computeQueueIndex = 1;
        }
        continue;
    }
}
```

# References
##### Main Notes
[[Listing the queue families]]
[[Checking for hardware suitability]]
#### Source Notes
[[Drawing a triangle]]