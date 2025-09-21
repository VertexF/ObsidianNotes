2025-09-21 10:47
Status: #adult 
Tags: [[vulkan]] [[vulkan instance]]
# Creating a vulkan instance

After creating the application struct, you will want to try and create an instance. A vulkan instance is the highest level object and come from the vulkan-1.dll itself. It is used to create everything else in vulkan and therefore should be the last things destroyed.

```c++
VkInstanceCreateInfo createInfo{};
	createInfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
	createInfo.pApplicationInfo = &applicationInfo; //The basic app set up struct
    
    VkInstance instance;
    vkCreateInstance(&createInfo, nullptr, &instance);
```

This is how pretty much everything is created in vulkan a struct then a create function. Destroying is very similar. 

It's very important that the code above is just an example and wouldn't fly in normal application because with no extensions the instance can't set up any rendering code and with no validation layers being set up you have no debugging.
# References
##### Main Notes
[[First Step for setting up Vulkan]]
[[Setting up extensions]]
[[What are validation layers]]
[[How to debug instance creation]]
[[Cleaning a Vulkan Instance]]
#### Source Notes
