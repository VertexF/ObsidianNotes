2025-10-26 10:11
Status: #baby 
Tags: [[vulkan]] [[vulkan graphic pipeline]]
# Setting up the colour blending

After the fragment shader has returned a colour we need to know what to do with the previous colour that was created. This is known as colour blending are there are two type of blending.
1) Mix the old and the new colours to make the final colour.
2) Combine both the old and the new values using a bitwise operation you provide.

When setting up colour blending you use 2 structs to do different colour blending for a global and per-framebuffer bases.
1) **VkPipelineColorBlendAttachmentState** is concerned with blending colour per-framebuffer attachment.
2) **VkPipelineColorBlendStateCreateInfo** is concerned with the global blending state.

```c++
VkPipelineColorBlendAttachmentState colourBlendAttachment{};
colourBlendAttachment.colorWriteMask = VK_COLOR_COMPONENT_R_BIT | VK_COLOR_COMPONENT_G_BIT | VK_COLOR_COMPONENT_B_BIT | VK_COLOR_COMPONENT_A_BIT;
```

For a zero blending for a given framebuffer you set things up like this.
```c++
colourBlendAttachment.blendEnable = VK_FALSE;
colourBlendAttachment.srcColorBlendFactor = VK_BLEND_FACTOR_ONE;
colourBlendAttachment.dstColorBlendFactor = VK_BLEND_FACTOR_ZERO;
colourBlendAttachment.colorBlendOp = VK_BLEND_OP_ADD;
colourBlendAttachment.srcAlphaBlendFactor = VK_BLEND_FACTOR_ONE;
colourBlendAttachment.dstAlphaBlendFactor = VK_BLEND_FACTOR_ZERO;
colourBlendAttachment.alphaBlendOp = VK_BLEND_OP_ADD;
```

For alpha blending you'll want to do things like this.
```c++
colourBlendAttachment.blendEnable = VK_TRUE;
colourBlendAttachment.srcColorBlendFactor = VK_BLEND_FACTOR_SRC_ALPHA;
colourBlendAttachment.dstColorBlendFactor = VK_BLEND_FACTOR_ONE_MINUS_SRC_ALPHA;
colourBlendAttachment.colorBlendOp = VK_BLEND_OP_ADD;
colourBlendAttachment.srcAlphaBlendFactor = VK_BLEND_FACTOR_ONE;
colourBlendAttachment.dstAlphaBlendFactor = VK_BLEND_FACTOR_ZERO;
colourBlendAttachment.alphaBlendOp = VK_BLEND_OP_ADD;
```

Over all the this is what the blending is doing when you set **.blendEnable** to VK_TRUE with this pseudocode 

```c++
if(blendEnable)
{
	finalColour.rgb = (srcColourBlendFactor * newColour.rgb) <colorBlendOp> (dstColorBlendFactor * oldColour.rgb);
	
	finalColour.a = (srcAlphaBlendFactor * newColour.rgb) <alphaBlendOp> (dstAlphaBlendFactor * oldColour.a);
}
else
{
	finalColour = newColour;
}

finalColour = finalColour & colourWriteMask;
```

When you want to achieve to alpha blending, you want to take the new colour and the old colour and based on opacity you multiply the alpha with new and old colours -1 for the old colour. This is effectively what alpha blending is doing you saw above.

```c++
finalColour.rgb = newAlpha * newColour + (1 - newAlpha) * oldColour;
finalColour.a = newAlpha.a;
```

**VkPipelineColorBlendStateCreateInfo** takes in a array of structures for all the framebuffers and allows you to set blend constants that you can use for blend factors in the above calculations. This is what this would look like.

```c++
VkPipelineColorBlendStateCreateInfo colourBlending{};
colourBlending.sType = VK_STRUCTURE_TYPE_PIPELINE_COLOR_BLEND_STATE_CREATE_INFO;
colourBlending.logicOpEnable = VK_FALSE;
colourBlending.logicOp = VK_LOGIC_OP_COPY; //optional
colourBlending.attachmentCount = 1;
colourBlending.pAttachments = &colourBlendAttachment;
colourBlending.blendConstants[0] = 0.f; //optional
colourBlending.blendConstants[1] = 0.f; //optional
colourBlending.blendConstants[2] = 0.f; //optional
colourBlending.blendConstants[3] = 0.f; //optional
```

If you need the second type of blending with the bitwise operation. You need to set the **.logicOpEnable** to VK_TRUE. You need to set the **.pAttachments** which takes the colour blend operation via the **VkPipelineColorBlendAttachmentState** to have a value for **.blendEnable** VK_TRUE for some of the framebuffers. The **.logicOp** is the logical bitwise operation. 

The operations that will be applied to the framebuffer colour components **.colorWriteMask** **VkPipelineColorBlendAttachmentState** from the colour struct. 
# References
##### Main Notes
#### Source Notes
[[Vulkan-Tutorial]]