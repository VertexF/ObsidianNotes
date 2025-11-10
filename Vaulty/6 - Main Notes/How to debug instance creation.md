2025-09-21 10:25
Status: #adult 
Tags: [[vulkan]] [[vulkan instance]] [[vulkan validation]]
# How to debug instance creation

You may not know that creation of the debug callback function from the debug util extension requires a successfully created vulkan instance to set up. So how do you debug instance creation?

Well you simply create the **VkDebugUtilsMessengerCreateInfoEXT** before the **VkInstanceCreateInfo** and extend **VkInstanceCreateInfo** with **VkDebugUtilsMessengerCreateInfoEXT** with the .pNext feature
```c++
        VkDebugUtilsMessengerCreateInfoEXT debugMessageInfo = {};
        debugMessageInfo.sType =                                VK_STRUCTURE_TYPE_DEBUG_UTILS_MESSENGER_CREATE_INFO_EXT;
        debugMessageInfo.pNext = nullptr;
        debugMessageInfo.pfnUserCallback = debugUtilsCallback;
        debugMessageInfo.messageSeverity =                       VK_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT |
VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT |
VK_DEBUG_UTILS_MESSAGE_SEVERITY_VERBOSE_BIT_EXT;
         debugMessageInfo.messageType =                          VK_DEBUG_UTILS_MESSAGE_TYPE_PERFORMANCE_BIT_EXT |            VK_DEBUG_UTILS_MESSAGE_TYPE_VALIDATION_BIT_EXT;
         debugMessageInfo.pUserData = nullptr;
         
VkInstanceCreateInfo createInfo{};
	createInfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
	createInfo.pNext = &debugMessageInfo;
	//..ect
```

So now with creation and deletion of the vulkan instance you have validation layers set to use the function callback function if something goes wrong for creation and deletion of the instance.
# References
##### Main Notes
[[Creating a vulkan instance]]
[[Creating the custom callback function for validation]]
[[Connecting custom debug function to Vulkan]]
[[Instance and device level validation]]
#### Source Notes
[[Drawing a triangle]]