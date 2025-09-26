2025-09-26 10:21
Status: #baby 
Tags: [[vulkan]] [[vulkan surface]] [[queue families]]
# Checking to see if presentation is supported

When checking for queue families you need to also check for presentation queue families too. It's possible for you to find graphics queue family and that queue family **NOT** support presentation.

To do this you need to check with the function **vkGetPhysicalDeviceSurfaceSupportKHR** this takes in the physical device, the surface you've created and check against the queue family index you've found to see if it has presentation support.

I wanted graphics, computer and presentation to all be using the same queue family index. This is for performance reasons.

```c++

VkBool32 surfaceSupport = VK_FALSE;
for (uint32_t familyIndex = 0; familyIndex < queueFamilyCount; ++familyIndex)
{
	VkQueueFamilyProperties queueFamily = queueFamilies[familyIndex];
	if (queueFamily.queueCount > 0 && queueFamily.queueFlags &                            (VK_QUEUE_GRAPHICS_BIT | VK_QUEUE_COMPUTE_BIT))
        {
            check(vkGetPhysicalDeviceSurfaceSupportKHR(physicalDevice, familyIndex, vulkanWindowSurface, &surfaceSupport));

                // If we have some how found a queue family 
                // that support graphics and compute commands 
                // but not presention we need to skip that one.
                if (surfaceSupport)
                {
                    vulkanMainQueueFamily = familyIndex;
                    break;
                }
            }
        }
```

You see here I'm looking for a queue family that has compute and only then checking if there is a presentation flag. If that's found with the **surfaceSupport** this become our **vulkanMainQueueFamily** which is used to create the main VkQueue where graphics and presentation commands run on.
# References
##### Main Notes
[[Listing the queue families]]
#### Source Notes
[[Vulkan-Tutorial]]