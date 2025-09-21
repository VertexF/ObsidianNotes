2025-09-21 09:40
Status: #adult  
Tags: [[vulkan]] [[vulkan validation]]
# Connecting custom debug function to Vulkan

To do this you need to do more than just create the struct and run the function to connect things up because this function needs to be dynamically loaded at run time.

The struct is pretty straight forward. Each struct argument tells Vulkan what messages to send to a custom debug function.
```c++
        VkDebugUtilsMessengerCreateInfoEXT creationInfo = {};
        creationInfo.sType =                                VK_STRUCTURE_TYPE_DEBUG_UTILS_MESSENGER_CREATE_INFO_EXT;
        creationInfo.pNext = nullptr;
        creationInfo.pfnUserCallback = debugUtilsCallback;
        creationInfo.messageSeverity =                       VK_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT |
VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT |
VK_DEBUG_UTILS_MESSAGE_SEVERITY_VERBOSE_BIT_EXT;
         creationInfo.messageType =                          VK_DEBUG_UTILS_MESSAGE_TYPE_PERFORMANCE_BIT_EXT |            VK_DEBUG_UTILS_MESSAGE_TYPE_VALIDATION_BIT_EXT;
         createInfo.pUserData = nullptr;
```

You can sent more than this if you want the details of these severity message and message types can be found in the [[Creating the custom callback function for validation]] . **pfnUserCallback** takes a custom debug function pointer which should follow the specification. **pUserData** is were you set up the pointer to be returned in a custom callback function, this can be left null.

You'll need to load the functions dynamically more for info on that go to [[Loading extended functions dynamically]] 

So lets look at the function arguments 
```c++
vkCreateDebugUtilsMessengerEXT(vulkanInstance, &debugMessenegerCreateInfo, vulkanAllocationCallbacks, &vulkanDebugUtilsMessenger);
```

Firstly is the vulkan instance, second is the struct we made earlier, third is the allocator and forth is the **VkDebugUtilsMessengerEXT** which tracks the life span of the connected custom debug function.

If you've done that right you should have a connected debug function up and running. It's important to note again this should be **ONLY** set up in debug builds.
# References
##### Main Notes
[[Dealing with VK_EXTENSION_NOT_PRESENT error]]
[[Creating the custom callback function for validation]]
[[Loading extended functions dynamically]]
#### Source Notes
[[Vulkan-Tutorial]]