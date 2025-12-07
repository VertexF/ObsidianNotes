2025-11-21 09:36
Status: #baby 
Tags: [[vulkan]] [[vulkan graphic pipeline]] [[vulkan descriptors]]
# Setting up the pipeline layout with a descriptor

To create a pipeline layout that uses descriptors you first need to have finished [[Creating a descriptor set layout]]  

```c++
VkPipelineLayout pipelineLayout;
VkPipelineLayoutCreateInfo pipelineLayoutCreateInfo{};
pipelineLayoutCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO;
pipelineLayoutCreateInfo.setLayoutCount = 1;
pipelineLayoutCreateInfo.pSetLayouts = &uboLayoutBinding;

if (vkCreatePipelineLayout(device, &pipelineLayoutCreateInfo, nullptr, &pipelineLayout) != VK_SUCCESS) 
{
	printf("Failed to create the layout pipeline.\n");
}
```

This is extremely similar to [[Setting up an empty pipeline layout]] but we are just adding in our descriptor set layout we created eariler.
# References
##### Main Notes
[[What are descriptors]]
[[Creating a descriptor set layout]] 
[[Setting up an empty pipeline layout]] 
#### Source Notes
[[Uniform buffers]]