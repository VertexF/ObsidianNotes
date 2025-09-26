2025-09-26 12:11
Status: #baby 
Tags: [[vulkan]] [[vulkan queue]]
# Creating a separate queue

Now the meat of this is already explained in [[Creating vulkan queues]] please go there for details about there creation. This will just be an addition to how these queues can be little different.

When listing queue families, I actually reject GPUs that don't have a graphics, compute and presentation family, you can see this here.
[[Checking to see if presentation is supported]] This means I can literally just make a "main" queue family and create them normally as one big VkQueue before device creation.

However if you've split up the presentation or any other type of queue family you'll need to create more than one **VkDeviceQueueCreateInfo** struct, add the list and count of them to **VkDeviceCreateInfo** before device creation.
# References
##### Main Notes
[[Creating vulkan queues]]
[[Checking to see if presentation is supported]]
[[Create the logical device]]
#### Source Notes
[[Vulkan-Tutorial]]