2025-11-21 09:40
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]]
# Creating a descriptor set layout

This will be a super basic example of how to use a uniform buffer object with a [[What is a descriptor set layout|descriptor set layout]] and a [[What are descriptor sets|descriptor set]] We are using this vertex shader to simply do MVP calculation on each vertex attribute.
```c++
#version 450

layout (binding = 0) uniform ModelData
{
	mat4 model;
	mat4 view;
	mat4 proj;
}modelData;

layout (location = 0) in vec2 inPosition;
layout (location = 1) in vec3 inColour; 

layout(location = 0) out vec3 fragColour;

void main()
{
	gl_Position = modelData.proj * modelData.view * vec4(inPosition, 0.0, 1.0);
	fragColour = inColour;
}
```

The `binding = 0` is similar to the `location = 0` directive. We will reference the binding directly in the pipeline set up with the descriptor set layout. We need to do this for every single location.

So lets create it
```c++
VkDescriptorSetLayoutBinding uboLayoutBinding{};
uboLayoutBinding.binding = 0;
uboLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
uboLayoutBinding.descriptorCount = 1;
```

**.binding** and **.descriptorType** for telling vulkan what we are using the descriptor for and at what binding. We are making our **.descriptorType** a **VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER** so a uniform buffer object.

The **.descriptorCount** tells vulkan how many values are in the array. For example, in skinning you can have a load of MVP matrices that are all different that need to be uploaded if you're doing animation on the CPU. For the simplest case of upload a 1 MVP we set this to 1.

```c++
uboLayoutBinding.stageFlags = VK_SHADER_STAGE_VERTEX_BIT;
```

This specified where it will be used. You can OR a bunch of these together or make it used in every shader step with **VK_SHADER_STAGE_ALL_GRAPHICS**.

```c++
uboLayoutBinding.pImmutableSamplers = nullptr;
```

These are used for image sampler related to descriptors.

All the descriptor set layouts are rolled into a **VkDescriptorSetLayout** type and this goes into
[[Setting up the pipeline layout with a descriptor|VkPipelineLayout]] which then gets rolled into a graphics pipeline. So lets create the **VkDescriptorSetLayout**

```c++
VkDescriptorSetLayoutCreateInfo layoutInfo{};
layoutInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_LAYOUT_CREATE_INFO;
layoutInfo.bindingCount = 1;
layoutInfo.pBindings = &uboLayoutBinding;

VkDescriptorSetLayout descriptorSetLayout;
if (vkCreateDescriptorSetLayout(device, &layoutInfo, nullptr, &descriptorSetLayout) != VK_SUCCESS)
{
    printf("Failed to create descriptor set layout");
}
```

**.pBindings** and **.bindingCount** are used to handle arrays but if you have 1 you just do the typical.
# References
##### Main Notes
[[Setting up the pipeline layout with a descriptor]]
[[Setting up an empty pipeline layout]]
[[What are descriptor sets]]
[[What is a descriptor set layout]]
#### Source Notes
[[Uniform buffers]]