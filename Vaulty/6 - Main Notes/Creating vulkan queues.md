2025-09-24 11:46
Status: #baby 
Tags: [[vulkan]] [[vulkan queue]]
# Creating vulkan queues

As the **VkQueue's** are automatically created when you create the logical device. You will need to go a fetch them with the VkQueue indices. 

To load them you'll want to run the **vkGetDeviceQueue** function. This uses the indices of the queue family, the queue index and the vulkan logical device.

```c++
vkGetDeviceQueue(vulkanDevice, computeQueueFamilyIndex, 0, &vulkanComputeQueue);
```

The arguments in order are:
- 1) The VkDevice
- 2) The family queue index value. 
- 3) The queue index
- 4) The VkQueue
The queue index is needed here if you have set up more than 1 VkQueue of the same type when you created the **VkDeviceQueueCreateInfo**. So if you decided you need 50 compute queues in the VkDeviceQueueCreateInfo struct you would need to set the 3rd argument to 49. (I could be wrong it might be 50 I'm not sure.)
# References
##### Main Notes
[[Setting up vulkan queues]]
[[Create the logical device]]
[[Creating a separate queue]]
#### Source Notes
[[Drawing a triangle]]