2025-09-24 08:57
Status: #baby 
Tags: [[vulkan]] [[logical device]]
# What is vulkan logical device

The logical device is the interface between the physical device and Vulkan commands, you can have more than one logical device if you need it. 

When creating the logical device you should have the **VkPhysicalDevice** and the queue family indices you've gotten from GPU. After you've done that you need to set up the **VkDeviceQueueCreateInfo** this with that struct you can create the queues with the given queue family index. Then you can create the queues with the **VkDevice** creation.
# References
##### Main Notes
[[What is a vulkan physical device]]
[[What are queue families]]
[[Setting up vulkan queues]]
#### Source Notes
[[Drawing a triangle]]