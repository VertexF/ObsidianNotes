2025-11-07 13:55
Status: #baby 
Tags: [[vulkan]] 
# Setting up the synchronisation objects

So to handle drawing to a screen we will need 2 semaphores and 1 fence.

```c++
VkSemaphore imageAvailableSemaphore;
VkSemaphore renderFinishSemaphore;
VkFence framesInFlight;

VkSemaphoreCreateInfo semaphoreInfo{};
semaphoreInfo.sType = VK_STRUCTURE_TYPE_SEMAPHORE_CREATE_INFO;

VkFenceCreateInfo fenceInfo{};
fenceInfo.sType = VK_STRUCTURE_TYPE_FENCE_CREATE_INFO;
fenceInfo.flags = VK_FENCE_CREATE_SIGNALED_BIT;

if (vkCreateSemaphore(device, &semaphoreInfo, nullptr, &imageAvailableSemaphore) != VK_SUCCESS)
{
		printf("Failed to create image available semaphore");
}

if (vkCreateSemaphore(device, &semaphoreInfo, nullptr, &renderFinishSemaphore) != VK_SUCCESS)
{
	printf("Failed to create render finished semaphore");
}

if (vkCreateFence(device, &fenceInfo, nullptr, &framesInFlight) != VK_SUCCESS)
{
	printf("Failed to create the fence.");
}
```

The only important part is the **VkCreateFenceInfo** **.flags** this is set to **VK_FENCE_CREATE_SIGNALED_BIT** meaning we set up the fence to be signalled from the start. You want to do this because we be waiting for the fence to be signled before we begin our next frame. Well what about the first frame? There wasn't a previous frame to wait for. If we don't set up the fence at the beginning we will dead locked immediately. 

Destroying them is pretty simple too.

```c++
vkDestroySemaphore(device, imageAvailableSemaphore, nullptr);
vkDestroySemaphore(device, renderFinishSemaphore, nullptr);
vkDestroyFence(device, framesInFlight, nullptr);
```
# References
##### Main Notes
[[What is synchronisation in vulkan]]
[[What are semaphores]]
[[What are fences]]
#### Source Notes
[[Vulkan-Tutorial]]
