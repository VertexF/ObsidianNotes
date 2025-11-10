2025-11-02 10:34
Status: #baby 
Tags: [[vulkan]] [[vulkan graphic pipeline]]
# Creating derived graphic pipelines

There are two new parameters **.basePipelineHandle** and **.basePipelineIndex** with these you can make graphics pipelines that are derived from an existing pipeline. The upside of this is that if they share common functionality it's faster to create a graphics pipeline, and it's faster to switch between child and parent graphics pipelines too.

To do this you either use **.basePipelineHandle** to refer to a graphics pipeline object or **.basePipelineIndex** to refer to a graphics pipeline with an index. 

For both the parent and the child graphics pipeline the **.flag** setting needs to be **VK_PIPELINE_CREATE_DERIVATIVE_BIT** of the VkGraphicsPipelineCreateInfo struct. The child needs all the mandatory paramters just like it's parent. 

If you want to do this via the **.basePipelineIndex** you have to create the parent and child together in one function call. If you want to use the **.basePipelineHandle** you have to create parent first then the child.

You have to use one **OR** the other. You can't use both. You have to turn off other version by setting **.basePipelineHandle** to VK_NULL_HANDLE or the **.basePipelineIndex** to -1

This is the set up using **.basePipelineIndex**

```c++
VkGraphicsPipelineCreateInfo pipelineInfo{};
childPipelineInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
childPipelineInfo.flags = VK_PIPELINE_CREATE_DERIVATIVE_BIT;
//... Set up stuff is here.
pipelineInfo.basePipelineHandle = VK_NULL_HANDLE;
pipelineInfo.basePipelineIndex = 1;

VkGraphicsPipelineCreateInfo childPipelineInfo{};
childPipelineInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
childPipelineInfo.flags = VK_PIPELINE_CREATE_DERIVATIVE_BIT;
//... Set up stuff is here.
childPipelineInfo.basePipelineHandle = VK_NULL_HANDLE;
childPipelineInfo.basePipelineIndex = pipelineInfo.basePipelineIndex - 1;

VkGraphicsPipelineCreateInfo structs[] = { pipelineInfo, childPipelineInfo };

VkPipeline pipelines[2]{};
if (vkCreateGraphicsPipelines(device, VK_NULL_HANDLE, 2, pipelines, nullptr, childPipeline) != VK_SUCCESS)
{
	assert(false);
}
```

You have to set the child's **.basePipelineIndex** to one minus the parent's **.basePipelineIndex** with the line `childPipelineInfo.basePipelineIndex = pipelineInfo.basePipelineIndex - 1;`

Then you create both pipelines using array's instead of seperately. 

Creation of the using **.basePipelineHandle** is much simpiler
```c++
VkGraphicsPipelineCreateInfo pipelineInfo{};
pipelineInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
pipelineInfo.flags = VK_PIPELINE_CREATE_ALLOW_DERIVATIVES_BIT;
//... All the pipeline creation
pipelineInfo.basePipelineHandle = VK_NULL_HANDLE;
pipelineInfo.basePipelineIndex = -1;

VkPipeline mainPipeline;
if (vkCreateGraphicsPipelines(device, VK_NULL_HANDLE, 1, &pipelineInfo, nullptr, &mainPipeline) != VK_SUCCESS) 
{
	assert(false);
}

VkGraphicsPipelineCreateInfo childPipelineInfo{};
childPipelineInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
childPipelineInfo.flags = VK_PIPELINE_CREATE_DERIVATIVE_BIT;
//... All the pipeline creation
childPipelineInfo.basePipelineHandle = mainPipeline;
childPipelineInfo.basePipelineIndex = -1;
	
VkPipeline childPipeline;
if (vkCreateGraphicsPipelines(device, VK_NULL_HANDLE, 1, &childPipelineInfo, nullptr, &childPipeline) != VK_SUCCESS)
{
	assert(false);
}
```

We actually don't care about the parent index or handle here, these can be ignored. Look at the **.basePipelineHandle** in the childPipelineCreateInfo and you can see it's refering **mainPipeline**. 

The value of the child **.basePipelineIndex** has to be -1.
# References
##### Main Notes
[[Creating the graphics pipeline]]
#### Source Notes
[[Drawing a triangle]]