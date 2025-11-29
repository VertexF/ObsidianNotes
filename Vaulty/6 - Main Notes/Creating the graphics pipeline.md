2025-11-02 10:26
Status: #baby 
Tags: [[vulkan]][[vulkan graphic pipeline]]
# Creating the graphics pipeline

To complete this you need to set up the **VkGraphicsPipelineCreateInfo** with all it's parameters linking back to the different creation structs you have made containing
- **Shader stages** These define the functionality of the programmable stages of the graphics pipeline.
- **Fixed function states** These define the fixed function graphics pipeline stuff.
- **Pipeline Layout** These define uniforms and the push constants you'll be using for your shaders.
- **Render pass**: These define how VkImage's should be handled by the different graphics pipeline stages.

Remember don't delete the shader modules with **vkDestroyShaderModule** before you create this graphics pipeline.

To start you set the shader modules.  
```c++
VkGraphicsPipelineCreateInfo pipelineInfo{};
pipelineInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
pipelineInfo.stageCount = 2;
pipelineInfo.pStages = shaderStages;
```

The number 2 defines how many shaders are in the flat array of **shaderStages**.

Then you have all the fixed function stuff
```c++
pipelineInfo.pVertexInputState = &vertexInputState;
pipelineInfo.pInputAssemblyState = &inputAssembly;
pipelineInfo.pViewportState = &viewportCreateInfo;
pipelineInfo.pRasterizationState = &rasteriser;
pipelineInfo.pMultisampleState = &multisamplingState;
pipelineInfo.pDepthStencilState = nullptr; //Optional
pipelineInfo.pColorBlendState = &colourBlending;
pipelineInfo.pDynamicState = &dynamicState;
```

Note **.pDepthStencilState** is the only graphics pipeline that's optional.

Then you want the pipeline layout.
```c++
pipelineInfo.layout = &pipelineLayout;
```

Then you want to add your render pass with the subpass index of 0.
```c++
pipelineInfo.renderPass = renderPass;
pipelineInfo.subpass = 0;
```

You can set things up to work with more than 1 render pass but they all have to be compatible with each other.

Then you can create everything with the **vkCreateGraphicsPipelines**

```c++
VkPipeline mainPipeline{};
if (vkCreateGraphicsPipelines(device, VK_NULL_HANDLE, 1, &pipelineInfo, nullptr, &mainPipeline) != VK_SUCCESS) 
{
	assert(false);
}
```

The rest of the arguments are as list
1) The logical device
2) The pipeline cache value which can be null.
3) The amount of **VkGraphicsPipelineCreateInfo** you are passing in.
4) The **VkGraphicsPipelineCreateInfo** themselves
5) The vulkan allocator callback
6) The final **VkPipeline**

The second argument which takes in a **VkPipelineCache** value can be really useful, if you need to recreate a pipeline you've already made before. For example, if you're running a game and you make all the graphics pipelines needed you actually save them to disk and load this caches next time saving pipeline create time.
# References
##### Main Notes
[[Creating derived graphic pipelines]]
**Setup**
>**Shaders**
>[[Compiling shaders with vulkan]]
>[[Reading in compiled shaders]]
>[[Creation of the shader module]]
>**Fixed Functions** 
>[[Setting up the dynamic states]]
>[[Setting up the vertex inputs]]
>[[Setting up the input assembly]]
>[[Setting up the viewports and scissors]]
>[[Setting up the rasteriser]]
>[[Setting up the multisampler]]
>[[Setting up the depth and stencil testing]]
>[[Setting up the colour blending]]
>**Pipeline Layout**
>[[Setting up an empty pipeline layout]]
>**Render pass**
>[[What is a render pass]]
>[[Setting up the colour attachment descriptions]]
>[[Setting up the attachment reference]]
>[[Setting up the subpass]]
>[[Setting up the render pass]]

#### Source Notes
[[Drawing a triangle]]