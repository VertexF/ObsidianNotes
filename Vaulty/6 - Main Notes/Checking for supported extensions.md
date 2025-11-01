2025-09-20 16:19
Status: #adult 
Tags: [[vulkan]] [[vulkan instance]]
# Checking for supported extensions

When you try to run the vkCreateInstance function you can run into the error **VK_ERROR_EXTENSION_NOT_PRESENT** you'll find your missing an extension. When you get this error it might be okay to exit the program if you're trying to support Window System Interface extension on a computer doesn't have it, like a server or something.

There are optional extensions that you might want to check for before adding to the list to see if they are available. You do with with the **vkEnumerateInstanceExtensionProperties**

You actually run this twice first to get the number of extensions and second to get the list of those numbered extensions.

```c++
uint32_t numInstanceExtensions;
vkEnumerateInstanceExtensionProperties(nullptr, &numInstanceExtensions, nullptr);
std::vector<VkExtensionProperties> extensions(numInstanceExtensions); 
vkEnumerateInstanceExtensionProperties(nullptr, &numInstanceExtensions, extensionsDebug);

for (size_t i = 0; i < numInstanceExtensions; ++i)
{
	//Check for extensions by accessing the flat extensions[i] to see if you're thing is supported
}
```

The first argument of **vkEnumerateInstanceExtensionProperties** is null so it's checking all the extensions on every layer, meaning that you can filter the search for a layer if you want.

Otherwise the last argument is null, you get the count. If it's not null the function expects to write into a flat array with a list of **VkExtensionProperties**

Then you can check the list in the for loop to see if the optional extension you want is part of the list, if it is add it to your extensions and vkCreateInstance so work fine.
# References
##### Main Notes
[[Creating a vulkan instance]]
[[Setting up extensions]]
[[Device level extensions]]
#### Source Notes
[[Vulkan-Tutorial]]
