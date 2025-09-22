2025-09-22 06:57
Status: #adult  
Tags: [[vulkan]] [[vulkan physical device]]
# Checking for hardware suitability

Not all GPUs are created equal. It's very common for people to have more than 1 GPU in their computer. To select one we need to look at the properties and the features of the GPU and select the one we care about.

To get the properties we use the function **vkGetPhysicalDeviceProperties** and the get the features we use **vkGetPhysicalDeviceFeatures**. 

The properties tell us things like what type of GPU it is discrete or integrated, what version of vulkan is supported and the name. The features tell us for that type of GPU what features does it support like does it support mesh shader, halfs and doubles.

Querying the GPUs based on the list of physical devices might look like this. Here we are checking if the GPU is discrete and support geometry shaders.

```c++
bool isDeviceSuitable(VkPhysicalDevice device)
{
	VkPhysicalDeviceProperties properties;
	VkPhysicalDeviceFeatures features;
	vkGetPhysicalDeviceProperties(hardwareDevice, &properties);
	vkGetPhysicalDeviceFeatures(hardwareDevice, &features);
	
	return properties.deviceType == VK_PHYSICAL_DEVICE_TYPE_DISCRETE_GPU && features.geometryShader;
}
```

A common way to select a good GPU is to use a scoring system that adds points based on the features that are available with a simple int. Then you just select the highest scoring GPU from the list.
# References
##### Main Notes
[[Listing the GPUs in Vulkan]]
[[What is a vulkan physical device]]
#### Source Notes
[[Vulkan-Tutorial]]