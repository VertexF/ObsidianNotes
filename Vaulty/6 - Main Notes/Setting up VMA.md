2026-07-25 09:36
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Setting up VMA

To use VMA you need to create the VMA allocator with a creation struct

```c++
VmaAllocatorCreateInfo allocatorInfo{};
allocatorInfo.flags = VMA_ALLOCATOR_CREATE_BUFFER_DEVICE_ADDRESS_BIT;
allocatorInfo.physicalDevice = vulkanPhysicalDevice;
allocatorInfo.device = vulkanDevice;
allocatorInfo.instance = vulkanInstance;

result = vmaCreateAllocator(&allocatorInfo, &VMAAllocator);
check(result);
```

You need the device, instance and physical device. Then you can return the successfully create     **VmaAllocator**.

The flag **VMA_ALLOCATOR_CREATE_BUFFER_DEVICE_ADDRESS_BIT** is only needed if you have the bufferDeviceAddress feature available. 

That's pretty much all you need to do for the set up but you can set up a common function that will be used with a **VmaVulkanFunctions** set up.

```c++
VmaVulkanFunctions vkFunctions{};
vkFunctions.vkGetInstanceProcAddr = vkGetInstanceProcAddr;
vkFunctions.vkGetDeviceProcAddr = vkGetDeviceProcAddr;
vkFunctions.vkCreateImage = vkCreateImage;

VmaAllocatorCreateInfo allocatorInfo{};
allocatorInfo.pVulkanFunctions = &vkFunctions;
```

This set up VMA so it used in these common function calls. However, I don't use this so I haven't tested it.
# References
##### Main Notes
[[Setting up a vulkan buffer]]
#### Source Notes
[[How to vulkan]]