2025-09-23 07:27
Status: #adult 
Tags: [[vulkan]] [[queue families]]
# What are queue families

Vulkan works with commands, you submit a command and works get done. When you submit a command it goes in a VkQueue. These however are created with VkQueueFamily indices and these come from the VkPhysicalDevice.

Each GPU you select will have a different number of queue families you can use. It's important to select a queue family that can create a queue that allows you to do the commands you want. For example, if you have a queue that only support memory transfers, you wouldn't be able to submit graphics and computer commands to it.

Remember a single queue family can have more than one queue type. It's common for GPUs to have a queue family with everything in including the graphics queue and a separate transfer queue. 
# References
##### Main Notes
[[Listing the queue families]]
[[What is a vulkan physical device]]
#### Source Notes
[[Vulkan-Tutorial]]