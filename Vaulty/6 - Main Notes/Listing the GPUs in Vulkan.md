2025-09-22 06:46
Status: #adult  
Tags: [[vulkan]] [[vulkan physical device]]
# Listing the GPUs in Vulkan

When you want to connect Vulkan to a physical device you need to see if there are GPUs to connect with in the first place. You can do this with the function call **vkEnumeratePhysicalDevices** this works exactly the same to the **vkEnumerateInstanceExtensionProperties** when [[Checking for supported extensions]]

```c++
uint32_t numPhysicalDevice;

vkEnumeratePhysicalDevices(vulkanInstance, &numPhysicalDevice, nullptr);

std::vector<VkPhysicalDevice> gpus(numPhysicalDevice);

vkEnumeratePhysicalDevices(vulkanInstance, &numPhysicalDevice, gpus);
```

Very simply you just start with a **vkEnumeratePhysicalDevices** with the third argument null to get the count then you run the function again with a flat array to get the list of VkPhysicalDevices.
# References
##### Main Notes
[[Checking for hardware suitability]]
[[What is a vulkan physical device]]
[[Checking for supported extensions]]
#### Source Notes
[[Drawing a triangle]]
