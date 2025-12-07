2025-10-26 09:57
Status: #baby 
Tags: [[vulkan]] [[vulkan graphic pipeline]] [[depth testing]]
# Setting up the depth and stencil testing

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
# References
##### Main Notes
[[Setting up the colour attachment descriptions]]
[[Setting up the attachment reference]]
[[Setting up the subpass]]
#### Source Notes
[[Drawing a triangle]]
[[Depth buffering]]