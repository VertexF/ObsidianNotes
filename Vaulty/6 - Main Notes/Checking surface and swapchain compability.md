2025-10-16 19:48
Status: #baby
Tags: [[vulkan]] [[vulkan swapchain]] [[vulkan surface]]
# Checking surface and swapchain compability

You need three things you need to check to make sure the surface and the swapchain work together:
- **VKSurfaceCapabilitiesKHR** which is the basic surface capabilities [[Querying surface capabilities]]
	- Min/max number of images the swapchain has.
	- Width/heigh of the swapchain images
- **VkSurfaceFormatKHR** which is the surface format containing [[Checking surface format]]
	- Pixel format
	- Colour space (SRGB)
- **VkPresentModeKHR** presentation mode which handles how the swapchain images in the queue. [[Selecting a presentation mode]]

Once you have all the right information you can correctly fill up the swapchain creation struct and create the swapchain.
# References
##### Main Notes
[[Querying surface capabilities]]
[[Checking surface format]]
[[Selecting a presentation mode]]
[[What is a swapchain]]
#### Source Notes
[[Drawing a triangle]]