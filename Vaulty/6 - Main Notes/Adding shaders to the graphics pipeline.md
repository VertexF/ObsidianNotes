2025-10-23 17:23
Status: #adult  
Tags: [[vulkan]] [[vulkan graphic pipeline]] [[vulkan shaders]]
# Adding shaders to the graphics pipeline

To connect shaders you have written to the graphics pipeline in vulkan so they can be used, you first need to have done the [[Creation of the shader module]]. Next you'll need to create the **VkPipelineShaderStageCreateInfo** struct for **ALL** shader to be added the pipeline.

This is pretty straight forward, this is just the code for the vertex shader module but any other shader type follows the same pattern.

```c++
VkPipelineShaderStageCreateInfo vertShaderModuleInfo{};
vertShaderModuleInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
vertShaderModuleInfo.stage = VK_SHADER_STAGE_VERTEX_BIT;
vertShaderModuleInfo.module = vertexShaderModule;
vertShaderModuleInfo.pName = "main";
vertShaderModuleInfo.pSpecializationInfo = nullptr; //optional
```

**.pName** is used to specify the entry point of the shader. You can have more than one shader that comes from a shader module so for the **specific** shader you want, you'll need to specify that shaders entry point.

**.pSpecializationInfo** is a value that helps you split up the shaders with a constant value that doesn't change. This means that you can do a `#if defined()` style operation in the shader. Meaning that things will get compiled out or in depending on the value, speeding up things at rendering time because you've removed an if statement.

To load this into the vulkan graphics pipeline all the **VkPipelineShaderStageCreateInfo** structs needs to be in a **flat** array.

```c++
VkPipelineShaderStageCreateInfo  shaderStages[] = { vertShaderModuleInfo, fragShaderModuleInfo };
```
# References
##### Main Notes
[[Creation of the shader module]]
[[What is the graphics pipeline]]
#### Source Notes
[[Drawing a triangle]]