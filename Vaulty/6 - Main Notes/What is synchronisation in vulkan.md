2025-11-07 13:28
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# What is synchronisation in vulkan

When you actually want to render a frame you need to do these things:
1) Wait for the previous frame to finish.
2) Acquire an image from the swapchain.
3) Record a command buffer which draws the scene onto that image.
4) Submit the recorded command buffer
5) Present the swapchain image.

This is the overview you need to keep in mind while other things are mention

With synchronisation everything explicit meaning that order of operation is up to us. We need to tell the driver what to do with all the synchronisation primitive. It's important to note that the GPU and CPU are asynchronous, meaning that a lot of vulkan functions will return before the work on the GPU is done.

There are 3 major things with need to synchronise
- Acquiring an image from the swap chain.
- Executing commands that draw onto the acquired image.
- Presenting images to the screen for presentation, then returning it to the swapchain.

This is great but the order of this function finishing on the GPU is undefined, so you'll need to wait for them to finish to get everything working correctly.
# References
##### Main Notes
[[What are semaphores]]
[[What are fences]]
[[What you need to synchronise a frame]]
#### Source Notes
[[Vulkan-Tutorial]]