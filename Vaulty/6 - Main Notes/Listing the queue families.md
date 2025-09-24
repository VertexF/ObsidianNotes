2025-09-23 07:34
Status: #adult 
Tags: [[vulkan]] [[queue families]]
# Listing the queue families

Listing queue families follows the same pattern as listening physical deceives and extensions. You want to pull out the count and then the **VkQueueFamilyProperties**.

This is all done with the **vkGetPhysicalDeviceQueueFamilyProperties**

```c++
uint32_t queueFamilyCount = 0;
vkGetPhysicalDeviceQueueFamilyProperties(vulkanPhysicalDevice, &queueFamilyCount, nullptr);

std::vector<VkQueueFamilyProperties> queueFamilies(queueFamilyCount);

vkGetPhysicalDeviceQueueFamilyProperties(vulkanPhysicalDevice, &queueFamilyCount, queueFamilies);
O```
# References
##### Main Notes
[[Checking for family queue suitability]]
[[What are queue families]]
[[Listing the GPUs in Vulkan]]
[[Setting up extensions]]
#### Source Notes
[[Vulkan-Tutorial]]