2025-11-07 13:48
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# What you need to synchronise a frame

To synchronise a frame you'll need a fence to pause the host, to make sure that we are rendering and presenting has been completed. Remember we can't rerecord command buffers this would overwrite the command buffer while it's being used.

You'll also need two semaphores to order the operations that are happening with in the swapchain and the rendering. One for acquiring the next image that's ready to be draw to. Another semaphore for making sure that we wait for things to finish rendering, before we present the image to the screen.

These are the steps you need to take for sychronisation and presentention of a frame:
1) [[Using a fence to synchronise the previous frame]]
2) [[Acquirng an image from the swapchain]]
3) [[Submitting the command buffer for rendering]] 
4) [[Presenting to the screen]]
5) [[Handling frames in flight]]
# References
##### Main Notes
[[What is synchronisation in vulkan]]
#### Source Notes
[[Drawing a triangle]]