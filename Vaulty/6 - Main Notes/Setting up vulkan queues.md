2025-09-24 10:27
Status: [[vulkan]] [[vulkan queue]]
Tags: #baby 
# Setting up vulkan queues

To create the logical device you need to create the queue creation structure for the queues you want to create. This is because **VkQueue's** are tightly coupled with the **VkDevice**

To able to do this you need to have the **VkQueueFamily** indices already and have decided out of these **VkQueueFamily** indices how many **VkQueue's** you need

It's important to note that you typically want 1 VkQueue for a given VkQueueFamily, by default. Having more than 1 creates **graphics driver overhead**. If you want to try and submit a lot jobs at the same time, you want to **multithread the creation of command buffers** and submit all the work to the main thread which is very fast for the CPU to do. 

 This is how you would do this for a **simple single VkQueueFamily** index.
```c++
VkDeviceQueueCreateInfo queueCreateInfo{};
queueCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
//This is our main index we found from going over the queue families eariler.
queueCreateInfo.queueFamilyIndex = mainQueueFamilyIndex;
//You can create more than 1 queue if you need it here.
queueCreateInfo.queueCount = 1;
```

You need to set the priorities of queues when they are created from a value from 1.f to 0.f. These have help order and schedule of the command buffer execution on that queue. Default should be 1.f if you're unsure, giving the command buffers created from the queue the highest priority. 

```c++
queueCreateInfo.pQueuePriorities = 1.f;
```
# References
##### Main Notes
Command Buffer note needed here.
[[Making GPU features usable]]
[[What is vulkan logical device]]
#### Source Notes
[[Vulkan-Tutorial]]