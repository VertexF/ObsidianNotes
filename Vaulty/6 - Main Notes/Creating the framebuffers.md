2025-11-02 12:30
Status: #baby 
Tags: [[vulkan]] [[vulkan framebuffer]]
# Creating the framebuffers

When you need to create a framebuffer you'll need
1) The render pass. 
2) The render passes attachment descriptions.
3) The swapchain image views 
4) The swapchain extents

If you have all them then you'll want to create framebuffers in array that is the size of the amount of swapchain images you have.

```c++
std::vector<VkFramebuffer> swapchainFramebuffers(swapchainImageViews.size());

for (uint32_t i = 0; i < swapchainImageViews.size(); ++i) 
{
	VkImageView attachments[] =
	{
		swapchainImageViews[i]
	};
	
	VkFramebufferCreateInfo framebufferInfo{};
	framebufferInfo.sType = VK_STRUCTURE_TYPE_FRAMEBUFFER_CREATE_INFO;
	framebufferInfo.renderPass = renderPass;
	framebufferInfo.attachmentCount = 1;
	framebufferInfo.pAttachments = attachments;
	framebufferInfo.width = swapchainExtent.width;
	framebufferInfo.height = swapchainExtent.height;
	framebufferInfo.layers = 1;
	
	if (vkCreateFramebuffer(device, &framebufferInfo, nullptr, &swapchainFramebuffers[i]) != VK_SUCCESS)
	{
		printf("Failed to create a framebuffer pass.");
	}
}
```

The framebuffer needs the render pass you're going to use a draw time in the **.renderPass**.

The **.pAttachments** are the swapchain images. Note we only have 1 attachment descriptions in the static array which described the swapchain colour image. Note this isn't the same attachment description that we used when [[Setting up the render pass]] this is the swapchain's VkImageView. 

We are making a new array here so if we have more attachment descriptions we can add them to the `attachments[]` array. The **.attachmentCount** is the totally number of attachments we have in our render pass and framebuffer.

The **.layers** refers to number layers in image arrays. Our swapchain images are simply only having a single image, so no array's here. Meaning we set the **.layer** to 1.

Deletion is pretty straight forward.

```c++
for (uint32_t i = 0; i < swapchainFramebuffers.size(); ++i)
{
	vkDestroyFramebuffer(device, swapchainFramebuffers[i], nullptr);
}
```
# References
##### Main Notes
[[What is a framebuffer]]
[[What is a render pass]]
[[Setting up the render pass]]
[[Setting up the colour attachment descriptions]]
[[What is a swapchain]]
[[Getting swapchain images]]
[[Creating the swapchain]]
#### Source Notes
[[Vulkan-Tutorial]]