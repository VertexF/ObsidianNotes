2025-11-07 15:23
Status: #baby 
Tags: [[vulkan]] [[vulkan subpass]]
# What are subpass dependencies

Subpasses dependencies are to do with memory transitions that happen within a render pass automatically depending on the operation and graphics pipeline stage. The subpass will transition as soon as they can.

If you don't set up the subpass dependencies on a subpass it will be executed in any order within the render pass, which isn't always an error. It's possible have a subpass that just needs to be executed at some undefined point within a render pass.

You have to be very careful, when you're using a render pass and setting up the sychronisation between the images with semaphore. The operations in the subpass need to happen at the correct time to match up with the semaphores signalling if you don't set up the dependencies you can end up undefined behaviour.

What if you only have 1 subpass within a render pass? Well it's important note that subpasses have self dependencies. Which might need to be set up if you say allowing the vertex shader to run before the swapchain has finished reading the image from the last frame and pausing operation until the swapchain has finish reading the image so you can write out the final colour. I do this in the {{Setting the sychronisation with the subpass dependencies}}
# References
##### Main Notes
[[What is a subpass]]
#### Source Notes
[[Render Passes in Vulkan]]