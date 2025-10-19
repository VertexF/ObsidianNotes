2025-10-16 17:05
Status: #baby 
Tags: [[vulkan]] [[vulkan swapchain]]
# What is a swapchain

A swapchain is how vulkan handles rendering images to the screen. It's a queue that contains VkImages that get swapped in when they need to be rastered by the screen monitor and swapped out to be reused to be draw on to.

You need to have a least set up the logical device to begin writing the swapchain.
# References
##### Main Notes
[[Setting up the swapchain extension]]
[[What is vulkan logical device]]
#### Source Notes
[[Vulkan-Tutorial]]