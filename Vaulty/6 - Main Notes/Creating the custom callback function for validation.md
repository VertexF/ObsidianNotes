2025-09-21 08:38
Status: #adult 
Tags: [[vulkan]] [[vulkan validation]]
# Creating the custom callback function for validation

When you create this function it needs to follow the vulkan specification to work. This is what a barebones one should look like.  

```c++
static VKAPI_ATTR VkBool32 VKAPI_CALL debugCallback (VkDebugUtilsMessageSeverityFlagBitsEXT messageSeverity,
VkDebugUtilsMessageTypeFlagsEXT messageType,
const VkDebugUtilsMessengerCallbackDataEXT* callbackData,
void* userData)
{
	printf("Validation error: %s", callbackData->pMessage);

	return VK_FALSE;
}
```

The function signature needs to match exactly like this. It needs to be **static**, have a return type of **VkBool32** and have **VKAPI_ATTRI** **VKAPI_CALL**. I've made a function without these last two things but I think that's a mistake on my end. You can call the function whatever you want.

The first argument tracks the severity of the message being received. These can be:
- `VK_DEBUG_UTILS_MESSAGE_SEVERITY_VERBOSE_BIT_EXT` Diagnostic info not meant for you.
- `VK_DEBUG_UTILS_MESSAGE_SEVERITY_INFO_BIT_EXT`General normal info of things working.
- `VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT` Likely a bug, but not fatal.
- `VK_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT` A fatal bug that needs to be solved.

The second argument track the type of message you'll be receiving which can be any of these:
- `VK_DEBUG_UTILS_MESSAGE_TYPE_GENERAL_BIT_EXT` This is for any old event unrelated to performance or the specification. 
- `VK_DEBUG_UTILS_MESSAGE_TYPE_VALIDATION_BIT_EXT` This is for when you've done something wrong against the specification.
- `VK_DEBUG_UTILS_MESSAGE_TYPE_PERFORMANCE_BIT_EXT`This is for when there is a potential performance issue.

The third argument is where the message itself is stored:
- `pMessage` The debug message in a null-terminated string (aka const char*)
- `pObjects` This is a flat array of objects that are related to the message.
- `objectCount` This contains the number of objects in the array.

The final argument is a pointer to whatever programmer has set up when setting up this extensions. This pointer can be a pointer to a class or struct object of your choosing or simply null, which is what I typically go with.

Finally this should **ALWAYS** return VK_FALSE. Returning anything else will result in a **VK_ERROR_VALIDATION_FAILED_EXT** which is meant for debugging the validation layers themselves.
# References
##### Main Notes
[[Why set up debug callbacks for vulkan validation]]
[[Connecting custom debug function to Vulkan]]
#### Source Notes
[[Drawing a triangle]]