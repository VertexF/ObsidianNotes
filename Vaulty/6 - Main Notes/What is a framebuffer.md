2025-11-02 12:25
Status: #baby 
Tags: [[vulkan]] [[vulkan framebuffer]]
# What is a framebuffer

The framebuffers set up is very much dependent on how you set up the render pass, as the render pass handles the framebuffers at draw time.

Attachments descriptions specified during the render pass are bound by wrapping them into a **VkFramebuffer** object. So the framebuffer contains all the VkImageView object that we created in the render pass via the **VkAttachmentDescription**. 

For each swapchain image you'll want to create a framebuffer for it. This will contain the render pass, subpass, attachment description and reference.

The framebuffer images after the set up are used within a command. When you're trying to use the render pass to do the drawing operations you need to give the render pass the framebuffer image it's going to use, from the list of framebuffers you have already created. 

The image index that selects the correct framebuffer is controlled by the swapchain. It tells us what image index is safe to draw to as the swapchain operates. This is all done at draw time.
# References
##### Main Notes
[[Creating the framebuffers]]
[[What is a render pass]]
[[Setting up the colour attachment descriptions]]
[[What is a swapchain]]
[[Getting swapchain images]]
[[Setting up the subpass]]
[[Setting up the attachment reference]]
#### Source Notes
[[Vulkan-Tutorial]]
