2025-09-26 10:21
Status: #child  
Tags: [[vulkan]] [[vulkan surface]] [[queue families]]
# Checking to see if presentation is supported

When checking for queue families you need to also check for presentation queue families too. It's possible for you to find graphics queue family and that queue family does **NOT** support presentation. You need if you want graphics to be displayed.

 To check if presentation is supported you use **vkGetPhysicalDeviceSurfaceSupportKHR** function this takes in the physical device, the surface you've created and check against the queue family index you've found to see if it has presentation support.

I wanted graphics, computer and presentation to all be using the same queue family index. This is for performance reasons.

```c++
VkBool32 supportPresentation = VK_FALSE;
for (uint32_t familyIndex = 0; familyIndex < queueFamilyCount; ++familyIndex)
{
	VkQueueFamilyProperties queueFamily = queueFamilies[familyIndex];
	if (queueFamily.queueCount > 0 && queueFamily.queueFlags &                            (VK_QUEUE_GRAPHICS_BIT | VK_QUEUE_COMPUTE_BIT))
        {
            check(vkGetPhysicalDeviceSurfaceSupportKHR(physicalDevice, familyIndex, vulkanWindowSurface, &supportPresentation));

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

You see here I'm looking for a queue family that has compute and graphics. Only then I check to see if presentation is supported with the flag **supportPresentation**. 

If **supportPresentation** is true then we set the **vulkanMainQueueFamily** (which is just an uint32_t) to the current index of the loop. This is later used to create the main VkQueue where graphics and presentation commands run from.
# References
##### Main Notes
[[What are queue families]]
[[Listing the queue families]]
#### Source Notes
[[Drawing a triangle]]