2025-11-28 08:06
Status: #baby 
Tags: [[vulkan]] [[vulkan image views]]
# Creating a texture image view

Right if you have completed the steps within [[Preparing the texture image]] you'll need to create the image view that surrounds the VkImage.

The VkImageView which contains all the metadata about the image this is almost exactly the same as [[Creating an image view]] apart from the fact we are using a VkImage from [[Preparing the texture image]] instead of the swapchain. So look at that for more details.

Make sure you clean up the image view.
# References
##### Main Notes
[[Creating an image view]]
[[Preparing the texture image]]
[[Destroying a image view]]
#### Source Notes
[[Texture mapping]]