2025-09-21 09:51
Status: #adult 
Tags: [[vulkan]]
# Loading extended functions dynamically

In vulkan when you want to use an extension you may need to load functions dynamically at run time. You will need a successfully created vulkan instance to be able to do this and a custom allocator if you want one but nullptr is fine.

I believe these functions come from the vulkan-1.dll

I will use the example of Vulkan Debug utils extension but the same pattern applies to any function being loaded, the only difference is the function name you're trying to load.

I would personally look up the documentation of the extension you're trying to use to make sure you get the names correct. So here is how you load the debug util extension function being loaded.

```c++
            PFN_vkCreateDebugUtilsMessengerEXT vkCreateDebugUtilsMessengerEXT =(PFN_vkCreateDebugUtilsMessengerEXT)vkGetInstanceProcAddr(vulkanInstance, "vkCreateDebugUtilsMessengerEXT");
```

Every dynamically loaded function follows the pattern of **PFN_** followed by the name of the function you want. So in this case we want to run the **vkCreateDebugUtilsMessengerEXT** function to create a debug message callback.

The **vkGetInstanceProcAddr** returns a void* so you need to case this into the function pointer you want. The arguments are just vulkan instance and the string name of the function you want to create. 

Then you can just run it like normal.
```c++
vkCreateDebugUtilsMessengerEXT(vulkanInstance, &debugMessenegerCreateInfo, vulkanAllocationCallbacks, &vulkanDebugUtilsMessenger);
```

It's important to note that destroy the objects that have been created from dynamically loaded functions, you also need to dynamically load the destroy equivalent. So in this case we created **vulkanDebugUtilsMessenger** object to destroy this we need to do the same thing again when destroy the object.

```c++
PFN_vkDestroyDebugUtilsMessengerEXT vkDestroyDebugUtilsMessengerEXT = (PFN_vkDestroyDebugUtilsMessengerEXT)vkGetInstanceProcAddr(vulkanInstance, "vkDestroyDebugUtilsMessengerEXT");

vkDestroyDebugUtilsMessengerEXT(vulkanInstance, vulkanDebugUtilsMessenger, vulkanAllocationCallbacks);
```

Following the same pattern we can destroy the **vulkanDebugUtilsMessenger** just like every other object in vulkan.
# References
##### Main Notes
[[Setting up extensions]]
#### Source Notes
