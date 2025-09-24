2025-09-24 11:12
Status: #baby 
Tags: [[vulkan]] [[logical device]]
# Create the logical device

So when creating the logical device you need to have the 3 things ready: 
- The queue indices [[Setting up vulkan queues]]
- The physical device features struct [[Making GPU features usable]]
- A list device level extensions you need [[Device level extensions]] we will talk more about this here.

To create the **VkDevice** you'll need to create the **VkDeviceCreateInfo** struct first which looks like this.
```c++
Array<const char*> deviceExtensions;
deviceExtensions.init(tempAllocator, 2);
        deviceExtensions.push(VK_KHR_SWAPCHAIN_EXTENSION_NAME);        deviceExtensions.push(VK_KHR_SHADER_DRAW_PARAMETERS_EXTENSION_NAME);
        
VkDeviceCreateInfo deviceCreateInfo{};
	deviceCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;
    deviceCreateInfo.queueCreateInfoCount = queueCount;
    deviceCreateInfo.pQueueCreateInfos = queueInfo;
    deviceCreateInfo.enabledExtensionCount = deviceExtensions.size;
    deviceCreateInfo.ppEnabledExtensionNames = deviceExtensions.data;
    deviceCreateInfo.pNext = &physicalDeviceFeature2;
```

First thing you need to do is make a list of device level extensions you need. This should be done in a flat array what's what `Array<const char*> deviceExtensions;` is doing. If you're rendering you will always need **VK_KHR_SWAPCHAIN_EXTENSION_NAME** no matter what.

Once you've done that you need to add them to the **VkDeviceQueueCreateInfo** struct at **.ppEnabledExtensionNames** and the count of how many extensions you have to **.enabledExtensionCount**

Next you'll need to list out the **VkDeviceQueueCreateInfo's** in a flat array. The array goes to **.pQueueCreateInfos** and the size of the array goes to **.queueCreateInfoCount** It's totally fine to just have 1 queue, just make sure the count is 1.

Finally you need to add the device features you require from the **vkGetPhysicalDeviceFeatures2** struct. You add that to the **.pNext** part of the struct.

Then you run the **vkCreateDevice** function like this.
```c++
result = vkCreateDevice(vulkanPhysicalDevice, &deviceCreateInfo, vulkanAllocationCallbacks, &vulkanDevice);

check(result);
```

The arguments are 
- 1) The VkPhysicalDevice 
- 2) The VkDeviceCreateInfo 
- 3) The VMA/allocator callbacks.
- 4) The VkDevice

# References
##### Main Notes
[[Cleaning up a vulkan logical device and queues]]
[[Setting up vulkan queues]]
[[Making GPU features usable]]
[[Device level extensions]]
#### Source Notes
[[Vulkan-Tutorial]]