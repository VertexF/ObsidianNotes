2025-10-19 19:18
Status: #baby 
Tags: [[vulkan]] [[vulkan shaders]]
# Creation of the shader module

This is going to part of the graphics pipeline as a **VkShaderModule**. You need to create with the good old creation struct pattern. 

Assuming you've aligned the char* pointer to a uint32_t* width by [[Reading in compiled shaders]] correctly you should okay. I'm only going to show the vertex shader but any shader has the same idea.

```c++
VkShaderModule vertexShader;

VkShaderModuleCreateInfo vertexCreateInfo{};
vertexCreateInfo.sType = VK_STRUCTURE_TYPE_SHADER_MODULE_CREATE_INFO;
vertexCreateInfo.codeSize = sizeOfVertexCode;
vertexCreateInfo.pCode = reinterpret_cast<const uint32_t*>(binVertCode);

if (vkCreateShaderModule(device, &vertexCreateInfo, nullptr, &vertexShader) != VK_SUCCESS) 
{
	printf("Failed to create the vertex shader module");
}
```

Once we have created the **VkShaderModule** we don't need to keep the binary shader code around. So don't forget to free that

```c++
free((void*)binVertCode);
binVertCode = nullptr;
```

The **VkShaderModule** is a thin wrapper that is going to be uploaded to the GPU to be compiled into GPU machine code. This means that it's part of the graphics pipeline meaning we can have **destroy** the **VkShaderModules** at the end of the creation of the graphics pipeline.

```c++
//... Above this comment is all the code needed to create a graphics pipeline.

vkDestroyShaderModule(device, vertexShaderModule, nullptr);
```

It's important to note that you can add more than one shader to a shader module.
# References
##### Main Notes
[[Reading in compiled shaders]]
[[What is the graphics pipeline]]
#### Source Notes
[[Vulkan-Tutorial]]