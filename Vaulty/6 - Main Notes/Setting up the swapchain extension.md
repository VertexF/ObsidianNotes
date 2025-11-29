2025-10-16 19:40
Status: #baby
Tags: [[vulkan]] [[vulkan swapchain]] [[vulkan physical device]]
# Setting up the swapchain extension

To start with you need to set up the swapchain extention at the physical device level. If this extension is not available on the physical devices you've select it's totally normal to close the application right there and then.

To check you would use the standard checking method. First we get the list of extensions for that physical device

```c++
uint32_t extensionCount = 0;
vkEnumerateDeviceExtensionProperties(physicalDevice, nullptr, &extensionCount, nullptr);
std::vector<VkExtensionProperties> availableExtensions(extensionCount);
vkEnumerateDeviceExtensionProperties(physicalDevice, nullptr, &extensionCount, availableExtensions.data());
```

Then we check if the **VK_KHR_SWAPCHAIN_EXTENSION_NAME** is in the list. If it is we check add to a list for a struct later on.

```c++
std::vector<const char*> deviceExtensions;
for (uint32_t i = 0; i < availableExtensions.size(); ++i) 
{
	if (strcmp(availableExtensions[i].extensionName, VK_KHR_SWAPCHAIN_EXTENSION_NAME))
	{
		deviceExtensions.push_back(VK_KHR_SWAPCHAIN_EXTENSION_NAME);
		continue;
	}
}
```
# References
##### Main Notes
[[What is a swapchain]]
#### Source Notes
[[Drawing a triangle]]