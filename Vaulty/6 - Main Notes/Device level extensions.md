2025-09-24 11:04
Status: #baby 
Tags: [[vulkan]] [[logical device]]
# Device level extensions

In vulkan there are instance level extensions [[Setting up extensions]] and device level extensions. There needs to be a distinction between the two because it's possible that you have a Windows machine and the GPU on that machine isn't able to render anything and is only used for compute.

So if you plan on rendering something you need to check for the swapchain extensions called the **VK_KHR_swapchain**. If this is present on the GPU then we are able to render images to a monitor. 
# References
##### Main Notes
Swapchain note is needed here
[[Setting up extensions]]
[[Instance and device level validation]]
#### Source Notes
[[Vulkan-Tutorial]]