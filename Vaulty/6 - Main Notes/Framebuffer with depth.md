2025-12-06 19:20
Status: #baby 
Tags: [[vulkan]] [[depth testing]]
# Framebuffer with depth

Next we need to make the depth image view we should have created [[Creating the depth image and view]] and add that as an attachment to the framebuffer, we already done this before with [[Creating the framebuffers]] now we have depth.

```c++
std::array<VkImageView, 2> attachments =
{
    swapchainImageViews[i],
    depthImageView
};

VkFramebufferCreateInfo framebufferInfo{};
framebufferInfo.sType = VK_STRUCTURE_TYPE_FRAMEBUFFER_CREATE_INFO;
framebufferInfo.renderPass = renderPass;
framebufferInfo.attachmentCount = uint32_t(attachments.size());
framebufferInfo.pAttachments = attachments.data();
framebufferInfo.width = swapchainExtent.width;
framebufferInfo.height = swapchainExtent.height;
framebufferInfo.layers = 1;

if (vkCreateFramebuffer(device, &framebufferInfo, nullptr, &swapchainFramebuffers[i]) != VK_SUCCESS)
{
    assert(false);
}
```

The swapchain image view is different for every swapchain but the same for the depth. As we have only one subpass for all of them we can write to the same depth buffer due to our semaphores.
# References
##### Main Notes
[[Creating the depth image and view]]
[[Creating the framebuffers]]
#### Source Notes
[[Depth buffering]]