# Reference https://vulkan-tutorial.com/Depth_buffering

##### 3D geometry without the depth buffer.
If you do everything needed to have a piece of 3D geometry added to your scene without a depth buffer, you'll have every piece of 3D geometry overlapping other piece of geometry in ways you wouldn't expect.

```c++
const std::vector<Vertex> vertices = 
{
    {{ -0.5f, -0.5f, 0.f }, { 1.f, 1.f, 1.f }, { 1.f, 0.f }},
    {{  0.5f, -0.5f, 0.f }, { 1.f, 1.f, 1.f }, { 0.f, 0.f }},
    {{  0.5f,  0.5f, 0.f }, { 1.f, 1.f, 1.f }, { 0.f, 1.f }},
    {{ -0.5f,  0.5f, 0.f }, { 1.f, 1.f, 1.f }, { 1.f, 1.f }},

    {{ -0.5f, -0.5f, -0.5f }, { 1.f, 1.f, 1.f }, { 1.f, 0.f }},
    {{  0.5f, -0.5f, -0.5f }, { 1.f, 1.f, 1.f }, { 0.f, 0.f }},
    {{  0.5f,  0.5f, -0.5f }, { 1.f, 1.f, 1.f }, { 0.f, 1.f }},
    {{ -0.5f,  0.5f, -0.5f }, { 1.f, 1.f, 1.f }, { 1.f, 1.f }}
};
```

As you can see with the vertices at -0.5f for the second piece of geometry but if you render this you get this result

![[overlapping_geometry.png]]

This happens because the second part of the geometry in the index buffer come after the first.

```c++
const std::vector<uint16_t> indices = 
{
    0, 1, 2, 2, 3, 0, //first
    4, 5, 6, 6, 7, 4  //second
};
```

You have two ways of solving the ordering of fragments so the depth is correct.
1) Sorting your indices front to back based on depth
2) Using a depth testing with a depth buffer.

Sorting geometry back to front with the index buffer is actually how you do order-dependent transparency, as the independent version is hard to do. What's most common and practical is sorting fragments with a depth buffer. 

A depth buffer is another attachment that will be part of the graphics pipeline. This holds the depth values for every position, like the colour attachment holds colour values for every position. Everytime the rasteriser produces a fragment, it will do a depth test to check if the new fragment is closer or further away from the previous one. If it **ISN'T** closer to the screen then the fragment gets discarded. Each time the fragment **IS** closer to the screen the fragment writes its depth value to the depth buffer.

You can modify the depth values in the fragment shader the same way you can modify the colour values.
##### Creating the depth image and view
We need to now create a depth attachment which is similar to when we were [[Creating a VkImage for texture mapping]] but this time we are going to make image depth instead of image colour. We need to create a depth attachment that's going to lieve for the entire life time of the swapchain. We can't use a simply do what we did with [[Getting swapchain images]] we will have to make our own.

Since this depth attachment is going to be part of the swapchain it needs to be re-created with the swapchain. So I'm going to wrap everything in function to make re-creation of the swapchain easier.

When creating the depth image the same resolutation as the colour attachment which will come from the swapchain extents, optimal tiling and device local memory. We also need to work out the special depth format we want to use. Again this is all similar to the [[Creating a VkImage for texture mapping]] 

Unlike with texture image we don't need to be specific to how we access the VkImage. If you remember the a texture image needs a VK_FORMAT_R8G8B8A8_SRGB to be able to be accessed in the fragment shader, but because we aren't going to be accessing the values directly in the fragment shader we have some flexibility. The formats you would want are:
- **VK_FORMAT_D32_SFLOAT** for 32-bit float for depth.
- **VK_FORMAT_F32_SFLOAT_8_UINT** for 32-bit signed flaot for depth and 8 bit stencil component
- **VK_FORMAT_D24_UNORM_S8_UINT** for 24-bit float for depth and 8 bit stencil component

Like with colour formats not all these depth formats are supported. So lets begin by writing a helper function to work out what's support or not.

```c++
static VkFormat findSupportedFormat(const std::vector<VkFormat>& candidtes, VkImageTiling tiling, VkFormatFeatureFlags features) 
{
    for (VkFormat format : candidtes) 
    {
        VkFormatProperties properties;
        vkGetPhysicalDeviceFormatProperties(physicalDevice, format, &properties);

        if (tiling == VK_IMAGE_TILING_LINEAR && (properties.linearTilingFeatures & features) == features) 
        {
            return format;
        }
        else if (tiling == VK_IMAGE_TILING_OPTIMAL && (properties.optimalTilingFeatures & features) == features)
        {
            return format;
        }
    }

    assert(!"No supported format based on the feature enum %d", features);
}
```

At a basic level this function loops through the vector **candidtes** and check to see if it support the either **VK_IMAGE_TILING_LINEAR** or **VK_IMAGE_TILING_OPTIMAL** with the `VkFormatProperties properties;` It's possible for the condition to be non-zero when doing the **AND** operation so we do an `==` operation to make sure that is the thing we want.

What are these features though? Well first you need to know what the tiling is this is dicussed in [[Creating a VkImage for texture mapping]] but basicly linear is for images you want in major-row order to change later and optimial is implementation defined order.
- **linearTilingFeatures** this tells us that format supports linear tiling.
- **optimalTilingFeatures** this tell us that format supports optimal tiling
- **bufferFeatures** Uses cases that are supported for buffers.

```c++
static VkFormat findDepthFormat() 
{
    return findSupportedFormat({VK_FORMAT_D32_SFLOAT, VK_FORMAT_D32_SFLOAT_S8_UINT, VK_FORMAT_D24_UNORM_S8_UINT},
        VK_IMAGE_TILING_OPTIMAL, VK_FORMAT_FEATURE_DEPTH_STENCIL_ATTACHMENT_BIT);
}
```

In this function we are taking the **findSupportedFormat** vector and ordering the formats we want the most to least and checking if any of them support **VK_FORMAT_FEATURE_DEPTH_STENCIL_ATTACHMENT_BIT** when we are **VK_IMAGE_TILING_OPTIMAL**..

Here we are very likely to skip over the stencil part of the depth buffer, I'm setting up the simplest depth buffer but please re-order things to your needs.

```c++
VkFormat depthFormat = findDepthFormat();

createImage(swapchainExtent.width, swapchainExtent.height, depthFormat, VK_IMAGE_TILING_OPTIMAL, VK_IMAGE_USAGE_DEPTH_STENCIL_ATTACHMENT_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, depthImage, depthImageMemory);
createImageView(depthImage, depthFormat, VK_IMAGE_ASPECT_DEPTH_BIT);
```

Now we create the depth image with the format, usage and memory properties. After that we can create the image view on top of the image that handles depth. 

This image is special we don't need to do any map memory operations because this will be cleared at the start of the render pass.
##### Explicitly transitioning the depth image
You don't have to explicitly transition the depth image, as it is done in the render pass however it might be useful knowledge to know how to do this. 

What we are doing here is changing how we handle image layout transition so this is an extension on [[Transitioning image memory for texture mapping]]

After completing all the stuff in [[Creating the depth image and view]]  you'll wanto 

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
##### Setting up the depth attachment description
This is extremely familar to [[Setting up the colour attachment descriptions]] but we do it for depth instead. So lets get started

```c++
VkAttachmentDescription depthAttachment{};
depthAttachment.format = findDepthFormat();
depthAttachment.samples = VK_SAMPLE_COUNT_1_BIT;
depthAttachment.loadOp = VK_ATTACHMENT_LOAD_OP_CLEAR;
depthAttachment.storeOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
depthAttachment.stencilLoadOp = VK_ATTACHMENT_LOAD_OP_DONT_CARE;
depthAttachment.stencilStoreOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
depthAttachment.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
depthAttachment.finalLayout = VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL;
```

The format is the **depth image itself**. Unlike the colour attachment description we don't care about **.storeOp** as it will not be used faster drawing has finished. Everything else explains itself.

This is extactly the same as [[Setting up the attachment reference]] and [[Setting up the subpass]] we just add depth now.
```c++
VkAttachmentReference depthAttachmentRef{};
depthAttachmentRef.attachment = 1;
depthAttachmentRef.layout = VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL;

VkSubpassDescription subpass{};
subpass.pipelineBindPoint = VK_PIPELINE_BIND_POINT_GRAPHICS;
subpass.colorAttachmentCount = 1;
subpass.pColorAttachments = &colourAttachmentRef;
subpass.pDepthStencilAttachment = &depthAttachmentRef;
```

Unlike the colour attachment, a subpass can only use 1 depth stencil attachment. As it doesn't really make sense to do depth test on multiple buffers.

```c++
VkSubpassDependency dependency{};
dependency.srcSubpass = VK_SUBPASS_EXTERNAL;
dependency.dstSubpass = 0;
dependency.srcStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT | VK_PIPELINE_STAGE_LATE_FRAGMENT_TESTS_BIT;
dependency.srcAccessMask = VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT;
dependency.dstStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT | VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT;
dependency.dstAccessMask = VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT | VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT;
```

Finally we need to extend the subpass dependencies to make sure there is no conflict between the transitioning of the depth image and it being cleared as of its load operation. We first access the depth buffer **VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT** stage which is were the load operation clears things. We need to specify the access mask to write with **VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT**, after the **VK_PIPELINE_STAGE_LATE_FRAGMENT_TESTS_BIT**.
##### Framebuffer with depth
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
##### Clear values
We now have multiple attachment with **VK_ATTACHMENT_LOAD_OP_CLEAR** we also need to specify multple clear values.

```c++
std::array<VkClearValue, 2> clearColour{};
clearColour[0] = { { 0.f, 0.535f, 1.f, 1.f } };
clearColour[1] = { { 1.f, 0.f } };
renderPassBeginInfo.clearValueCount = uint32_t(clearColour.size());
renderPassBeginInfo.pClearValues = clearColour.data();
```

The range for regular perspective depth testing is 0.0 to 1.0 were 1.f lies at the far view plane and 0.f lies at the near view plane. The initial value so the **1.f** should be at the furthest possible depth which is 1.f. For reverse perspective it would be 0.f.
##### Setting up the depth and stencil testing
The setting up of the depth stencil state for the graphics pipeline is set up with the **VkPipelineDepthStencilStateCreateInfo**

```c++
VkPipelineDepthStencilStateCreateInfo depthStencil{};
depthStencil.sType = VK_STRUCTURE_TYPE_PIPELINE_DEPTH_STENCIL_STATE_CREATE_INFO;
depthStencil.depthTestEnable = VK_TRUE;
depthStencil.depthWriteEnable = VK_TRUE;
```

**.depthTestEnable** tells vulkan that new fragments should be discarded if they are less than/greater or equal to depending on the perspsective direction. The **.depthWriteEnable** is enabled to write the fragment to the depth buffer if the fragment passes.

```c++
depthStencil.depthCompareOp = VK_COMPARE_OP_LESS;
```

This is the conversional test, which states that if the value is lower the fragment is closer. Remember with regular perspective the furthest value is 1.f. 

```c++
depthStencil.depthBoundsTestEnable = VK_FALSE;
depthStencil.minDepthBounds = 0.f; //optional
depthStencil.maxDepthBounds = 1.f; //optional
```

If you enable this feature only fragments that are within a range which is less than the total fragment will be kept. For the most part you don't want this, unless you know you need this type of optimisation.

```c++
depthStencil.stencilTestEnable = VK_FALSE;
depthStencil.front = {}; //optional
depthStencil.back = {}; //optional
```

The 3 field configure the stencil buffer operation. If you want to use this feature you'll want to make that you format the depth/stencil image to contain a stencil component.
##### Resizing the window with a depth buffer
You need to recreate the depth buffer on a window resize because the depth buffer resolution should change with the windows

```c++
static void recreateSwapchain()
{
    vkDeviceWaitIdle(device);

    vkGetPhysicalDeviceSurfaceCapabilitiesKHR(physicalDevice, surface, &surfaceCapabilities);
    swapchainExtent = surfaceCapabilities.currentExtent;

    if(swapchainExtent.width == 0 || swapchainExtent.height == 0) 
    {
        return;
    }

    cleanupSwapchain();

    createSwapchain();
    createImageViews();
    createDepthResources();
    createFramebuffers();
}
```

The function **createDepthResources** contains the code for the [[Creating the depth image and view]] 
##### Cleaning up depth resources
Just like with colour images you'll need to clean up resouces 
```c++
vkDestroyImageView(device, depthImageView, nullptr);
vkDestroyImage(device, depthImage, nullptr);
vkFreeMemory(device, depthImageMemory, nullptr);
```

##### Setting up reverse perspective for your depth buffer
When you follow tutorials online you'll find they do everything in the standard approach but the problem with with the regular depth buffer is that the precision of floating point values isn't infinite and we need to decide were the precision is going to be used for our depth buffer.

the precision of the depths is 1/z were z is the z the world space depth. This will map to 0, and 1. If you have a far plane and a near plane N and F you have this equation for the depth value d

**d = N(1/z) + F**

Since F and N are constants the d is mapped directly with 1/z

Lets map a float 32-bit value to a given depth value

![[regular_depth_z.png]]

You can see for each depth value we lose tonne of values near the asymptote. The reason here is that the 0 part of the y-axis has a lot more values when the graph line exploding to infinity but what if we reverse the y-axis? Well you get this

![[reverse_Z_depth.png]]

The depth values increase a 0 very quickly but this is were the line is levelling out to a constant value at the far plane. This massively improves the precision within the depth buffer. Espcially if you use 32-bit floats instead of 24 byte ints. This is called reverse-Z perspective!

You do need to do things differently for
- [[Setting up the depth and stencil testing]]
- [[Clear depth values]]

Firstly we need to make sure that we use the right perspective in our cglm calculation. You will want to define the macro 
```c++
#define CGLM_FORCE_DEPTH_ZERO_TO_ONE
#include <cglm/struct/vec2.h>
#include <cglm/struct/vec3.h>
#include <cglm/struct/affine.h>
#include <cglm/struct/cam.h>
```

This begin things off right with having our range from 0 to 1, if you don't define this the range is -1, 1 which we aren't interested in. 

Next you'll need to make the perspective matrix with the opposite far planes and near plane values. Since 0.f is going to be the far plane we need to reverse the order with this.

```c++
glms_perspective(glm_rad(45.f), swapchainExtent.width / (float)swapchainExtent.height, 100.f, 1.f);
```

Here **100.f** is the near plane and **1.f** is the far plane. Think about what we said before, the depth value is worked out to basically be 1/z and z is the world space depth. If we reverse the z values then 1/z which works out the depth value is reversed which is what we want.

Next in the graphics pipeline you need to change what the **.depthCompareOp** tests are.
```c++
depthStencil.depthCompareOp = VK_COMPARE_OP_GREATER_OR_EQUAL;
```

Why is the **VK_COMPARE_OP_GREATER_OR_EQUAL**? Well remember our Z values are reversed so it's actually the larger values that are closer to the near plane. So we want to keep fragments that are that are larger as they are closer to the near plane now. It's also true that **VK_COMPARE_OP_GREATER_OR_EQUAL** is the opposite to **VK_COMPARE_OP_LESS** so just by that alone it tells us it's want we want.

Finally we need to clear the depth values. Remember for the clear depth value the we start everything at the far plane so nothing get initially rejected. Well now it need to 0 and not 1. Since 0 is at the far plane now.

```c++
std::array<VkClearValue, 2> clearColour{};
clearColour[0] = { { 0.f, 0.535f, 1.f, 1.f } };
clearColour[1] = { { 0.f, 0 } };
renderPassBeginInfo.clearValueCount = uint32_t(clearColour.size());
renderPassBeginInfo.pClearValues = clearColour.data();
```

If you do that everything so render like you're in normal perspective but now you have a lot more depth values to work with between the far and near plane.