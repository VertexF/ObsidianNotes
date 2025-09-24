2025-09-24 07:01
Status: [[vulkan]] [[logical device]]
Tags: #baby 
# Making GPU features usable

When you want to use some feature such as dynamic rendering or using uint8_t in shaders you need to turn them on before you create the logical device.

How this works is you chain structures together with the .pNext of each feature. So if you want features that have been core in Vulkan 1.1 you will need to add that + anything else into the .pNext chain.

Once you've done that the chain of structs need to connect to the main feature request struct called **VkPhysicalDeviceFeatures2** guy should be final .pNext connection and it stops there.

Then you run the function **vkGetPhysicalDeviceFeatures2** with the first argument being the **VkPhysicalDevice** and the second argument being **VkPhysicalDeviceFeatures2** struct.

```c++
VkPhysicalDeviceFeatures2 physicalDeviceFeature2{};
physicalDeviceFeature2.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_FEATURES_2;

VkPhysicalDeviceVulkan11Features vulkan11Features{}; vulkan11Features.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_VULKAN_1_1_FEATURES;
void* currentPnext = &vulkan11Features;

VkPhysicalDeviceDynamicRenderingFeaturesKHR dynamicRenderingFeatures {};
dynamicRenderingFeatures.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_DYNAMIC_RENDERING_FEATURES_KHR;
	//This bool check comes from when we queried the hardware for the supported features.
    if (dynamicRenderingExtensionPresent)
    {
        dynamicRenderingFeatures.pNext = currentPnext;
        currentPnext = &dynamicRenderingFeatures;
    }

physicalDeviceFeature2.pNext = currentPnext;
vkGetPhysicalDeviceFeatures2(vulkanPhysicalDevice, &physicalDeviceFeature2);
```
# References
##### Main Notes
[[Checking for hardware suitability]]
#### Source Notes