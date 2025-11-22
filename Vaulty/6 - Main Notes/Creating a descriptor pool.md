2025-11-21 19:01
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]]
# Creating a descriptor pool

You can't just make descriptor sets, they are similar to command buffer, they need a descriptor pool to work.

```c++
VkDescriptorPoolSize poolSize{};
poolSize.type = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
poolSize.descriptorCount = imageCount;

VkDescriptorPoolCreateInfo poolDescriptorCreateInfo{};
poolDescriptorCreateInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_POOL_CREATE_INFO;
poolDescriptorCreateInfo.poolSizeCount = 1;
poolDescriptorCreateInfo.pPoolSizes = &poolSize;
```

First thing we need to is specify with **VkDescriptorPoolSize** which states how many descriptor set we want with **.descriptorCount** we are going create one descriptor set per frame so we will use the **imageCount**. Next we want to tell vulkan what the descriptor set will be handling using **.type** I want uniform buffers in my descriptor sets so I set this to **VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER** 

Next we pass that over to the **VkDescriptorPoolCreateInfo** and begin setting that up.

```c++
poolDescriptorCreateInfo.maxSets = imageCount;
```

We also need to specify the amount the **MAX** amount of descriptor sets. Again we know how many we want to allocate so this it set the **imageCount**.

You can set this optional **.flag** with something
```c++
poolDescriptorCreateInfo.flags = VK_DESCRIPTOR_POOL_CREATE_FREE_DESCRIPTOR_SET_BIT;
```

This tells the descriptor set if individual descriptors can be freed or not. If you're just using the simplest set up which is just simply creating them and **NOT** touching them apart from the data underneth then you **DON'T** need this. I left to 0.

```c++
VkDescriptorPool descriptorPool;
if (vkCreateDescriptorPool(device, &poolDescriptorCreateInfo, nullptr, &descriptorPool) != VK_SUCCESS)
{
    printf("Failed to create descriptor pool");
}
```

Then you can simply create your descriptor pool.
# References
##### Main Notes
[[How to use a descriptor set]]
#### Source Notes
[[Uniform buffers]]