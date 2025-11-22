2025-11-08 10:50
Status: #baby 
Tags: [[vulkan]]
# Handling frames in flight

Frames in flight is to do with how many frames are currently processing at the same time. In newer version of vulkan you can't just handle 1 frame at a time if your swapchain image count is more than 1.

Earlier on when you had extracted the minimum and maximum amount of swapchain images through checking it's capability. We can that we had this number of images.

```c++
uint32_t imageCount = surfaceCapabilities.minImageCount + 1;
```

This is the totally number of frame in flight you should have. 

Since we want to process these frames at the same time we also need to duplicated the command buffer and the synchronisation object.

This is what creation would look like the command buffer.
```c++
const int maxFramesInFlight = imageCount;

std::vector<VkCommandBuffer> commandBuffers(maxFramesInFlight);
VkCommandBufferAllocateInfo allocateInfo{};
allocateInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO;
allocateInfo.commandPool = commandPool;
allocateInfo.level = VK_COMMAND_BUFFER_LEVEL_PRIMARY;
allocateInfo.commandBufferCount = uint32_t(commandBuffers.size());

if (vkAllocateCommandBuffers(device, &allocateInfo, commandBuffers.data())) 
{
	printf("Failed to create a command buffer.");
}

std::vector<VkSemaphore> imageAvailableSemaphore(maxFramesInFlight);
std::vector<VkSemaphore> renderFinishSemaphore(maxFramesInFlight);
std::vector<VkFence> framesInFlight(maxFramesInFlight);
```

Since you know how to use flat arrays I'm sure you figure out the rest of allocate and creating semaphores and fences in a loop.

To do the rest we need to keep track of the current frame at run time. 

```c++
uint32_t currentFrame = 0;
bool running = true;
while(running)
{
	///... All event handling code here.
	
	vkWaitForFences(device, 1, &framesInFlight[currentFrame], VK_TRUE, UINT64_MAX);
	vkResetFences(device, 1, &framesInFlight[currentFrame]);

	//... All other drawing code here.

	currentFrame = (currentFrame + 1) % maxFramesInFlight;
}
```


The **currentFrame** should be set to 0 and index into command buffers, semaphores and fences as each frame needs it's own seperate synchronisation objects and command buffer. The last line caps the the current to cycle through 0, to n, where n is equal to how many images are in the swapchain.
# References
##### Main Notes
[[Setting up the command pool]]
[[What are command buffers]]
[[What is synchronisation in vulkan]]
[[What are semaphores]]
[[What are fences]]
#### Source Notes
[[Drawing a triangle]]