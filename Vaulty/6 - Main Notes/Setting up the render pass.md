2025-11-01 08:57
Status: #baby 
Tags: [[vulkan]] [[render pass]]
# Setting up the render pass

When you have finished:
- [[Setting up the colour attachment descriptions]]
- [[Setting up the attachment reference]]
- [[Setting up the subpass]]

You can now create a render pass with the **VkRenderPassCreateInfo** this struct takes in the struct
- **VkAttachmentDescription**
- **VkSubpassDescription**
It can take more than 1 of each types so if you need more subpasses and attachment descriptions to pair with them you'll use a flat array to store and link them to the struct.

If all goes well you'll end up with this

```c++
VkRenderPass renderPass;
VkRenderPassCreateInfo renderPassCreateInfo{};
renderPassCreateInfo.sType = VK_STRUCTURE_TYPE_RENDER_PASS_CREATE_INFO;
renderPassCreateInfo.attachmentCount = 1;
renderPassCreateInfo.pAttachments = &colourAttachment;
renderPassCreateInfo.subpassCount = 1;
renderPassCreateInfo.pSubpasses = &subpass;

if (vkCreateRenderPass(device, &renderPassCreateInfo, nullptr, &renderPass) != VK_SUCCESS) 
{
	printf("Failed to create a render pass.");
}
```

Destruction is simple, you just destroy **VkRenderPass** at some point before the VkDevice

 ```c++
vkDestroyRenderPass(device, renderPass, nullptr);
```
# References
##### Main Notes
[[Setting up the colour attachment descriptions]]
[[Setting up the attachment reference]]
[[Setting up the subpass]]
#### Source Notes
[[Vulkan-Tutorial]]