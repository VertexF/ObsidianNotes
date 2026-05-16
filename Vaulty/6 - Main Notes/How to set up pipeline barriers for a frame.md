2026-05-11 08:20
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# How to set up pipeline barriers for the start of the frame

Here is how you correctly set up a pipeline barrier at the beginning the of the frame. You need to do. You need to handle both colour and depth. The colour needs to be handled for both and this can't happen only on swapchain recreation.

```c++
VkImageMemoryBarrier2 barrierColour{};
barrierColour.sType = VK_STRUCTURE_TYPE_IMAGE_MEMORY_BARRIER_2;
barrierColour.oldLayout = VK_IMAGE_LAYOUT_UNDEFINED;
barrierColour.newLayout = VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL;
barrierColour.srcStageMask = VK_PIPELINE_STAGE_2_COLOR_ATTACHMENT_OUTPUT_BIT;
barrierColour.srcAccessMask = VK_ACCESS_NONE;
barrierColour.dstStageMask = VK_PIPELINE_STAGE_2_COLOR_ATTACHMENT_OUTPUT_BIT;
barrierColour.dstAccessMask = VK_ACCESS_COLOR_ATTACHMENT_READ_BIT | VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT;
barrierColour.srcQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
barrierColour.dstQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
barrierColour.image = vulkanSwapchainImages[vulkanImageIndex];
barrierColour.subresourceRange.baseMipLevel = 0;
barrierColour.subresourceRange.levelCount = 1;
barrierColour.subresourceRange.baseArrayLayer = 0;
barrierColour.subresourceRange.layerCount = 1;
barrierColour.subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;

Texture* depthTextureLocal = accessTexture(depthTexture);

VkImageMemoryBarrier2 barrierDepth{};
barrierDepth.sType = VK_STRUCTURE_TYPE_IMAGE_MEMORY_BARRIER_2;
barrierDepth.oldLayout = VK_IMAGE_LAYOUT_UNDEFINED;
barrierDepth.newLayout = VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL;
barrierDepth.srcStageMask = VK_PIPELINE_STAGE_2_LATE_FRAGMENT_TESTS_BIT;
barrierDepth.srcAccessMask = VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT;
barrierDepth.dstStageMask = VK_PIPELINE_STAGE_2_EARLY_FRAGMENT_TESTS_BIT;
barrierDepth.dstAccessMask = VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT;
barrierDepth.srcQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
barrierDepth.dstQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
barrierDepth.image = depthTextureLocal->vkImage;
barrierDepth.subresourceRange.baseMipLevel = 0;
barrierDepth.subresourceRange.levelCount = 1;
barrierDepth.subresourceRange.baseArrayLayer = 0;
barrierDepth.subresourceRange.layerCount = 1;
barrierDepth.subresourceRange.aspectMask = VK_IMAGE_ASPECT_DEPTH_BIT;

VkImageMemoryBarrier2 barriers[] = { barrierColour, barrierDepth };

VkDependencyInfo barrierColourDependencyInfo{};
barrierColourDependencyInfo.sType = VK_STRUCTURE_TYPE_DEPENDENCY_INFO;
barrierColourDependencyInfo.imageMemoryBarrierCount = 2;
barrierColourDependencyInfo.pImageMemoryBarriers = barriers;

vkCmdPipelineBarrier2(commandBuffer->vkCommandBuffer, &barrierColourDependencyInfo);
```

The **.oldLayout** is undefined in both because we 
1) We don't care
2) We don't know what the layout is going to be.

For each swapchain image you need to make sure you pause wait for **.srcStageMask** 

# References
##### Main Notes
#### Source Notes
