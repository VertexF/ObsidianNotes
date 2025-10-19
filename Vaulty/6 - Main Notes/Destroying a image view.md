2025-10-19 12:29
Status: #baby 
Tags: [[vulkan]] [[vulkan image views]]
# Destroying a image view

Unlike VkImage's which are automatically destroyed with the VkLogicalDevice, you need to destroy the VkImageViews we you no longer need them. 

You need to do this before the logical device is destroy.

```c++
	vkDestroyImageView(device, imageView, nullptr);
```
# References
##### Main Notes
[[Creating an image view]]
#### Source Notes
[[Vulkan-Tutorial]]
