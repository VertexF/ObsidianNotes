2025-11-07 13:48
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# What you need to synchronise a frame

To synchronise a frame you'll need a fence to pause the host, to make sure that we are rendering and presenting has been completed. Remember we can't rerecord command buffers this would overwrite the command buffer while it's being used.

You'll also need two semaphores to order the operations that are happening with in the swapchain and the rendering. One for acquiring the next image that's ready to be draw to. Another semaphore for making sure that we wait for things to finish rendering, before we present the image to the screen.

These are the steps you need to take for synchronisation and presentation of a frame:
1) [[Using a fence to synchronise the previous frame]]
2) [[Acquiring an image from the swapchain]]
3) [[Submitting the command buffer for rendering]] 
4) [[Presenting to the screen]]
5) [[Handling frames in flight]]

Frame in flight index is incremented and reset to zero. Swapchain image index is returned by vkAcquireNextImageKHR.

Frame in flight and swapchain images are different things. They could be anything, it's possible for example to have 2 frames in flight and 4 swapchain images. Meaning that you should **NEVER** mix up your current frame with swapchain image index. This is because the swapchain images are dependent on the presentation engine in the swapchain.

It's also important to point out that the swapchain will increment the swapchain images index itself. Don't touch it.

If you want you can use timeline semaphores to simplify things a little as timeline semphores are a super set of binary semphores. It just allows you to have less to keep track of as you increment a number by 1 and that's used to keep things in sync. 

If you use dynamic rendering the synchronisation process can simplify things, it's recommended on Desktop GPUs this is because it removes **VkFrameBuffer** and **VkRenderPass** from the process of synchronisation. Synchronisation1 and Synchronisation2 are actually pretty similar and don't change much. So if possible use Synchronisation2 since you'll be moving forward with time.
# References
##### Main Notes
[[The theory  of synchronising a frame]]
#### Source Notes
[[Drawing a triangle]]