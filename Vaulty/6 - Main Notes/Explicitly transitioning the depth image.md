2025-12-06 19:04
Status: #baby 
Tags: [[vulkan]] [[depth testing]]
# Explicitly transitioning the depth image

You don't have to explicitly transition the depth image, as it is done in the render pass however it might be useful knowledge to know how to do this. 

What we are doing here is changing how we handle image layout transition so this is an extension on [[Transitioning image memory for texture mapping]]

After completing all the stuff in [[Creating the depth image and view]]  you'll want to do this.

First we need to transition the image memory layout to **VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL**
```c++
transitionImageLayout(depthImage, depthFormat, VK_IMAGE_LAYOUT_UNDEFINED, VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL);
```

The first thing we need to do within the **transitionImageLayout** function to support depth images is to select the right aspect mask for our subresource
```c++
if (newLayout == VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL)
{
    barrier.subresourceRange.aspectMask = VK_IMAGE_ASPECT_DEPTH_BIT;

    if (hasStencilComponent(format))
    {
        barrier.subresourceRange.aspectMask |= VK_IMAGE_ASPECT_STENCIL_BIT;
    }
}
else
{
    barrier.subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
}
```

Here we are making sure that we are either both depth or/and stencil or just colour. The **hasStencilComponent** check if the format has stencil.

```c++
else if (oldLayout == VK_IMAGE_LAYOUT_UNDEFINED && newLayout == VK_IMAGE_LAYOUT_DEPTH_ATTACHMENT_OPTIMAL) 
{
    barrier.srcAccessMask = 0;
    barrier.dstAccessMask = VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_READ_BIT | VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT;

    sourceStage = VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT;
    destinationStage = VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT;
}
```

Here we are have another `else if` added to the **transitionImageLayout**. Here we are starting at the top of the pipe and moving to the **VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT** were the transition has to happen. Since we will be reading the depth test at **VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT** and writing to the depth image **VK_PIPELINE_STAGE_LATE_FRAGMENT_TESTS_BIT**. Meaning that the aspect masks need to be at depth stencil read and write.
# References
##### Main Notes
[[Creating the depth image and view]]
[[Transitioning image memory for texture mapping]]
[[Creating the depth image and view]]
#### Source Notes
[[Depth buffering]]