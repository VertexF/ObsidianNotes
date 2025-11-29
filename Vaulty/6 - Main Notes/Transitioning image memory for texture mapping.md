2025-11-27 19:02
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]]
# Transitioning image memory for texture mapping

The idea behind a **image memory barrier** is that it's used to synchronise and ensure that we are in the correct format for an operation to happen. For example, coping data over to a VkImage needs to be safe before a copy happens. Not just operation but also moving things like images to a different that are **VK_SHARING_MODE_EXCLUSIVE** queues also need these barriers.

Firstly I've abstracted the starting and ending of command buffer that are used for copying data. 
This is all covered in the [[Copying buffer data over]] note. Note we are using a graphics command pool here.

```c++
static void transitionImageLayout(VkImage image, VkFormat format, VkImageLayout oldLayout, VkImageLayout newLayout) 
{
    VkCommandBuffer commandBuffer = beginOneTimeCommandBuffer(commandPool);

    VkImageMemoryBarrier barrier{};
    barrier.sType = VK_STRUCTURE_TYPE_IMAGE_MEMORY_BARRIER;
    barrier.oldLayout = oldLayout;
    barrier.newLayout = newLayout;

    endOneTimeCommandBuffer(commandBuffer, commandPool, mainQueue);
}
```

So here we begin by setting up the **VkImageMemoryBarrier** which will do the transition from on barrier to another. 

```c++
barrier.srcQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
barrier.dstQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
```

If you are using barriers that have transfer queue ownership you need to set these 2 fields up. You need to set them to **VK_QUEUE_FAMILY_IGNORED** if you don't care, these can't be 0.

```c++
barrier.image = image;
barrier.subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
barrier.subresourceRange.baseMipLevel = 0;
barrier.subresourceRange.levelCount = 1;
barrier.subresourceRange.baseArrayLayer = 0;
barrier.subresourceRange.layerCount = 1;
```

Then we set up the subresourceRange and the image. The subresourceRange is to do with the what part of the image we are interested same as the buffer we have done before. Since we aren't using mipmaps then **.levelCount** and **.layerCount** is equal to 1.

```c++
barrier.srcAccessMask = 0;
barrier.dstAccessMask = 0;
```

These barriers are for access masks before and after. This is exactly the same as the [[Setting the synchronisation with the subpass dependencies|semaphores]]  and what they wait on. So if you need to wait for part of the graphics pipeline to finish before you can do a transfer operation then you'll set things up these. Be aware pipeline stages and access masks aren't the same thing.

I'm gonna show the actual function call we will to create our image barrier
```c++
VKAPI_ATTR void VKAPI_CALL vkCmdPipelineBarrier(
    VkCommandBuffer                             commandBuffer,
    VkPipelineStageFlags                        srcStageMask,
    VkPipelineStageFlags                        dstStageMask,
    VkDependencyFlags                           dependencyFlags,
    uint32_t                                    memoryBarrierCount,
    const VkMemoryBarrier*                      pMemoryBarriers,
    uint32_t                                    bufferMemoryBarrierCount,
    const VkBufferMemoryBarrier*                pBufferMemoryBarriers,
    uint32_t                                    imageMemoryBarrierCount,
    const VkImageMemoryBarrier*                 pImageMemoryBarriers);
```

2) **srcStageMask** is everything that needs to happen before the barrier can be used.
3) **dstStageMask** the things that can happen after a certain operation has happend.

For example, lets say you want to sample something from the fragment shader, then you'll want to wait until the **VK_ACCESS_TRANSFER_WRITE_BIT** before transitioning the image memory layout, then we can transition the image memory layout to something the fragment shader can read, which needs to happen before **VK_ACCESS_SHADER_READ_BIT**.

4)  **dependencyFlags** has 2 states **0** or **VK_DEPENDY_BY_REGION_BIT**. The region bit setting turns the barrier into a per-region condition. Meaning that when **VK_DEPENDY_BY_REGION_BIT** is set vulkan will start reading parts of the resource that have been written so far.

The rest deal with array's of pipeline barriers types
5) **memoryBarrierCount** is the amount of **pMemoryBarriers** you have
6) **pMemoryBarriers** is the actual flat array of **VkMemoryBarrier** types
7) **bufferMemoryBarrierCount** is the amount of **pBufferMemoryBarriers** you have
8) **pBufferMemoryBarriers** is the actual flat array of **VkBufferMemoryBarrier**
9) **imageMemoryBarrierCount** is the amount of **pImageMemoryBarriers** you have
10) **pImageMemoryBarriers** is the actual flat array of **VkImageMemoryBarrier**

So lets build up the pipeline stages and the access flags
```c++
VkPipelineStageFlags sourceStage = VK_PIPELINE_STAGE_FLAG_BITS_MAX_ENUM;
VkPipelineStageFlags destinationStage = VK_PIPELINE_STAGE_FLAG_BITS_MAX_ENUM;

if (oldLayout == VK_IMAGE_LAYOUT_UNDEFINED && newLayout == VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL) 
{
    barrier.srcAccessMask = 0;
    barrier.dstAccessMask = VK_ACCESS_TRANSFER_WRITE_BIT;

    sourceStage = VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT;
    destinationStage = VK_PIPELINE_STAGE_TRANSFER_BIT;
}
else if (oldLayout == VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL && newLayout == VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL)
{
    barrier.srcAccessMask = VK_ACCESS_TRANSFER_WRITE_BIT;
    barrier.dstAccessMask = VK_ACCESS_SHADER_READ_BIT;

    sourceStage = VK_PIPELINE_STAGE_TRANSFER_BIT;
    destinationStage = VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT;
}
else 
{
    printf("Not support memory image layout transition");
}
    
vkCmdPipelineBarrier(commandBuffer, sourceStage, destinationStage, 0, 0, nullptr, 0, nullptr, 1, &barrier);
```

This code is meant to 2 state changes:
1) Transitioning an image access mask from undefined to be transferable and the stages we are going to wait for are **VK_PIPELINE_STAGE_TRANSFER_BIT**. All to do a copy from a buffer to an image.
2) Tranitioning an image access mask from transfer writable to shader read bit and the stages are going to wait for are **VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT** All to read the image in the fragment shader.

**VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT** and **VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT** are both fake parts of the pipeline but put us before everything begins and after everything has ended for our pipeline stage. It's also true that **VK_ACCESS_TRANSFER_WRITE_BIT** is a pseudo-stage like the other 2 but this makes sure we are at the point were things are transfered within the graphics pipeline.

Note sometimes you want **VK_IMAGE_LAYOUT_GENERAL** for special cases here [[Introduction to texture mapping]]

When we run the **vkCmdPipelineBarrier** we are only interesting in **VkImageMemoryBarrier** so most of the rest is null. We just need to set the source and destination stages for our graphics pipeline to be waited on.

A quick note on **vkCmdPipelineBarrier** has to be done on a graphics queue.
# References
##### Main Notes
[[What are semaphores]]
[[Setting the synchronisation with the subpass dependencies]]
[[Setting up and running the vkCmdDraw command]]
[[Creating a VkImage for texture mapping]]
[[Introduction to texture mapping]]
[[Copying buffer data over]] 

#### Source Notes
[[Texture mapping]]