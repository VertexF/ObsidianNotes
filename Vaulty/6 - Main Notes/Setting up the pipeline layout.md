2025-10-26 16:37
Status: #baby 
Tags: [[vulkan]] [[vulkan graphic pipeline]]
# Setting up the pipeline layout

This is to do with the the **uniform** and **push constant**. These are like global variable for shaders they are often used for transformation matrices and textures samples. 

You have to create a **VkPipelineLayout** you do this with a creation struct. Now I'm going to create a **VkPipelineLayoutCreateInfo** that has no layout data in it, but look up what you need and set things accordingly.

```c++
VkPipelineLayout pipelineLayout;
VkPipelineLayoutCreateInfo pipelineLayoutCreateInfo{};
pipelineLayoutCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO;
pipelineLayoutCreateInfo.setLayoutCount = 0;
pipelineLayoutCreateInfo.pSetLayouts = nullptr;
pipelineLayoutCreateInfo.pushConstantRangeCount = 0;
pipelineLayoutCreateInfo.pPushConstantRanges = nullptr;

if (vkCreatePipelineLayout(device, &pipelineLayoutCreateInfo, nullptr, &pipelineLayout) != VK_SUCCESS) 
{
	assert(false);
	printf("Failed to create the layout pipeline.\n");
}
```

You have to clean up the **VkPipelineLayout** object before the VkDevice is destroyed.

```c++
vkDestroyPipelineLayout(device, pipelineLayout, nullptr);
```
# References
##### Main Notes
[[What is the graphics pipeline]]
#### Source Notes
[[Drawing a triangle]]