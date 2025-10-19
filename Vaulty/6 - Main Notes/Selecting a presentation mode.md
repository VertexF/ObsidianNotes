2025-10-17 13:27
Status: #baby 
Tags: [[vulkan]] [[vulkan swapchain]]
# Selecting a presentation mode

The presentation mode is how the swapchain handles it's VkImage's with the monitor vertical blank in it's queue. Selecting a different presentation mode can completely change the framerate and add or remove v-tearing so be careful with this one.

The vertical blank period is the amount of time between the last scan line hitting the bottom of the screen and returning to the top again.

- **VK_PRESENT_MODE_IMMEDIATE** - this mode updates the monitor as fast as possible regardless if it's finished doing it's vertical blank. Meaning that you can get v-tearing 
- **VK_PRESENT_MODE_FIFO_KHR** - has to be supported. With this mode in when a VkImage has been submitted to the queue the swapchain tries to put it in the display area. If there an image already there because the monitor is still in the middle of doing it's vertical blank, the swapchain simply waits. This is the v-sync option.
- **VK_PRESENT_MODE_RELAXED_KHR** -  This is only differs from **FIFO** if the application is late with drawing and the queue is empty at the vertical blank. Instead of waiting for the cycle to go through we just submit the VkImage right away, which may cause v-tearing. 
- **VK_PRESENT_MODE_MAILBOX_KHR** - If the application is faster than the vertical blank we just update the queue with a newer VkImage reducing latency. 

Again you have to select a valid mode that the hardware supported **VK_PRESENT_MODE_MAILBOX_KHR** is best for the most part if it's available otherwise we fall back to **VK_PRESENT_MODE_FIFO_KHR**

First we query in the same way as normal.
```c++
uint32_t presentationModeCount = 0;
vkGetPhysicalDeviceSurfacePresentModesKHR(physicalDevice, surface, &presentationModeCount, nullptr);
std::vector<VkPresentModeKHR> presentationModes(presentationModeCount);
vkGetPhysicalDeviceSurfacePresentModesKHR(physicalDevice, surface, &presentationModeCount, presentationModes.data());
```

Then we check for the one we want.
```c++
VkPresentModeKHR presentationMode = VK_PRESENT_MODE_MAX_ENUM_KHR;
for (uint32_t i = 0; i < presentationModes.size(); ++i)
{
	if (presentationModes[i] == VK_PRESENT_MODE_MAILBOX_KHR)
	{
		presentationMode = VK_PRESENT_MODE_MAILBOX_KHR;
		break;
	}

	presentationMode = VK_PRESENT_MODE_FIFO_KHR;
}
```
# References
##### Main Notes
[[Checking surface and swapchain compability]]
#### Source Notes
[[Vulkan-Tutorial]]