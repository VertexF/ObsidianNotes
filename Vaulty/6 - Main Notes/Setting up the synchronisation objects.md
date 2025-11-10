2025-11-07 13:55
Status: #baby 
Tags: [[vulkan]] 
# Setting up the synchronisation objects

So to handle drawing to a screen we will need 2 semaphores and 1 fence.

```c++
std::vector<VkSemaphore> imageAvailableSemaphore(maxFramesInFlight);
std::vector<VkSemaphore> renderFinishSemaphore(maxFramesInFlight);
std::vector<VkFence> framesInFlight(maxFramesInFlight);

VkSemaphoreCreateInfo semaphoreInfo{};
semaphoreInfo.sType = VK_STRUCTURE_TYPE_SEMAPHORE_CREATE_INFO;

VkFenceCreateInfo fenceInfo{};
fenceInfo.sType = VK_STRUCTURE_TYPE_FENCE_CREATE_INFO;
fenceInfo.flags = VK_FENCE_CREATE_SIGNALED_BIT;

for (uint32_t i = 0; i < maxFramesInFlight; ++i)
{
	if (vkCreateSemaphore(device, &semaphoreInfo, nullptr, &imageAvailableSemaphore[i]) != VK_SUCCESS)
	{
			printf("Failed to create image available semaphore");
	}

	if (vkCreateSemaphore(device, &semaphoreInfo, nullptr, &renderFinishSemaphore[i]) != VK_SUCCESS)
	{
		printf("Failed to create render finished semaphore");
	}

	if (vkCreateFence(device, &fenceInfo, nullptr, &framesInFlight[i]) != VK_SUCCESS)
	{
			printf("Failed to create the fence");
	}
}
```

The only important part is the **VkCreateFenceInfo** **.flags** this is set to **VK_FENCE_CREATE_SIGNALED_BIT** meaning we set up the fence to be signalled from the start. You want to do this because we be waiting for the fence to be signled before we begin our next frame. Well what about the first frame? There wasn't a previous frame to wait for. If we don't set up the fence at the beginning we will dead locked immediately. 

Destroying them is pretty simple too.

```c++
for (uint32_t i = 0; i < maxFramesInFlight; ++i)
{
	vkDestroySemaphore(device, imageAvailableSemaphore[i], nullptr);
	vkDestroySemaphore(device, renderFinishSemaphore[i], nullptr);
	vkDestroyFence(device, framesInFlight[i], nullptr);
}
```
# References
##### Main Notes
[[What is synchronisation in vulkan]]
[[What are semaphores]]
[[What are fences]]
[[Handling frames in flight]]
#### Source Notes
[[Drawing a triangle]]
