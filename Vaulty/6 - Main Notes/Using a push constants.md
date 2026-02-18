2026-02-07 09:37
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]]
# Using a push constants

These are a fantastic way to record changes to objects in a scene very efficently. They are easier set up than a descriptor set, but they have a size limit.

The set up for these is pretty straight forward you need to set up the range, and then tell the **VkPipelineLayoutCreateInfo** struct how many push constants you're using with the range you set up before.
```c++
VkPushConstantRange range{};
range.stageFlags = VK_SHADER_STAGE_VERTEX_BIT;
range.offset = 0;
range.size = 4;

VkPipelineLayoutCreateInfo pipelineLayoutInfo{};
pipelineLayoutInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO;
pipelineLayoutInfo.pSetLayouts = vkLayout;
pipelineLayoutInfo.setLayoutCount = creation.numActiveLayouts;
pipelineLayoutInfo.pushConstantRangeCount = 1;
pipelineLayoutInfo.pPushConstantRanges = &range;
```

The **VkPushConstantRange** is information based on the size of the push constants, offset and shader it's used in. The **.size** has to match the size of what within the push constants. If I look inside the shader you'll want to match this.

```c
layout(push_constant, std140) uniform entityIndex
{
    uint index;
};
```

In this case a uint is 4 bytes. However you could have many more bytes for a given type that you would need to set.

To use this you would need to record the command at the right time.

```c++
for (uint32_t i = 0; i < totalDucks; ++i)
{
    uint32_t offset = 0;
    uint32_t size = 4;
    vkCmdPushConstants(vkCommandBuffer, vkPipelineLayout, VK_SHADER_STAGE_VERTEX_BIT, offset, size, &i);
    
    //...other stuff
}
```

The last element here `&i` is the thing we are writing and uploading to the shader. The size and offset the same, as when we set then up. It's important to know that you need to have the **VkPipelineLayout** as part of the command structure.
# References
##### Main Notes
[[Push constant with offset data]]
[[What is a descriptor set layout]]
[[What are descriptors]]

#### Source Notes
[[Stream lessons]]