2025-10-17 13:44
Status: #baby 
Tags: [[vulkan]] [[vulkan swapchain]]
# Creating the swapchain

So it's important to note that the code here needs to be handled carefully. Assuming you've selected the structs you need to parse with these notes [[Querying surface capabilities]],
[[Checking surface format]] and [[Selecting a presentation mode]]

You now need to fill up the struct, this is how you start if off.

```c++
	VkSwapchainCreateInfoKHR createSwapchain{};
	createSwapchain.sType = VK_STRUCTURE_TYPE_SWAPCHAIN_CREATE_INFO_KHR;
	createSwapchain.surface = surface;
	createSwapchain.minImageCount = imageCount;
	createSwapchain.imageFormat = swapchainSurface.format;
	createSwapchain.imageColorSpace = swapchainSurface.colorSpace;
	createSwapchain.imageExtent = swapchainExtent;
	createSwapchain.imageArrayLayers = 1;
	createSwapchain.imageUsage = VK_IMAGE_USAGE_COLOR_ATTACHMENT_BIT;
```

**.imageArrayLayers** is set to one unless you want to do stereoscopic 3D. If you're writing directly to the images you use **VK_IMAGE_USAGE_COLOR_ATTACHMENT_BIT**. If you are writing to an image first then transfering it to **VK_IMAGE_USAGE_TRANSFER_DST_BIT**. In my engine I've OR-ed them together.

```c++
	//We are going to skip the code that actually gets the queue indices to keep the code simple.
	uint32_t presentationQueueFamily = 0;
	uint32_t graphicsQueueFamilyExclusive = UINT32_MAX;
	
	//... have gotten the queue family indices 
	
	std::vector<uint32_t> queueFamilyProps;
	queueFamilyProps.push_back(presentationQueueFamily);
	
	bool graphicAndPresentationAreSeparete = false; 
	if(graphicsQueueFamilyExclusive != UINT32_MAX)
	{
		queueFamilyProps.push_back(graphicsQueueFamilyExclusive);
		graphicAndPresentationAreSeparete = true;
	}

	if (queueFamilyProps.size() > 1 && graphicAndPresentationAreSeparete)
	{
		createSwapchain.imageSharingMode = VK_SHARING_MODE_CONCURRENT;
		createSwapchain.queueFamilyIndexCount = queueFamilyProps.size();
		createSwapchain.pQueueFamilyIndices = queueFamilyProps.data();
	}
	else 
	{
		createSwapchain.imageSharingMode = VK_SHARING_MODE_EXCLUSIVE;
		createSwapchain.queueFamilyIndexCount = 0; //optinal
		createSwapchain.pQueueFamilyIndices = nullptr; //optional
	}
```

So we have the image sharing mode set up for **VK_SHARING_MODE_CONCURRENT** if the presentation and graphics queue families are separate and is slower.  **VK_SHARING_MODE_EXCLUSIVE** means that the presentation queue and the graphics queue are the same thing, which is faster. 

```c++
createSwapchain.preTransform = surfaceCapabilities.currentTransform;
createSwapchain.compositeAlpha = VK_COMPOSITE_ALPHA_OPAQUE_BIT_KHR;
```

We need to current transform **VkSurfaceCapabilitiesKHR** structure. Then we want to set the **.compositeAlpha** to the value of **VK_COMPOSITE_ALPHA_OPAQUE_BIT_KHR** this means that the images are opaque. You can actually make swapchain transparent with the other windows.

```c++
createSwapchain.presentMode = presentationMode;
createSwapchain.clipped = VK_TRUE;
```

Then you set the presentation mode, which you should've already done. The **.clipped** needs to be VK_TRUE if you don't care about the pixels of the images if a window gets in the way of your game. You might really do care if you need consistency for each frame. 

```c++
createSwapchain.oldSwapchain = VK_NULL_HANDLE;
```

When you resize the window you need to reconstruct the swapchain. So when you are reconstructing the swapchain you need to give the old swapchain to the **.oldSwapchain** for optimisation reason.

Finally you can make the swapchain with the struct and the logical device.

```c++
VkSwapchainKHR swapchain;
vkCreateSwapchainKHR(device, &createSwapchain, nullptr, &swapchain);
	
vkDestroySwapchainKHR(device, swapchain, nullptr);
```
# References
##### Main Notes
[[Querying surface capabilities]]
[[Checking surface format]]
[[Selecting a presentation mode]]
#### Source Notes
[[Drawing a triangle]]