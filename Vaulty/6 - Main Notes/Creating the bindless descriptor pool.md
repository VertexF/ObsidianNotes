2026-02-27 08:48
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]] [[vulkan textures]]
# Creating the bindless descriptor pool

If you have enabled bindless, you need to create the descriptor pools differently than non-bindless. The idea here is we create a descriptor pool that supports updating things like textures **AFTER** they are bound.

```c++
VkDescriptorPoolSize poolSizeBindless[] =
{
	{ VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER, MAX_BINDLESS_RESOURCES },
	{ VK_DESCRIPTOR_TYPE_STORAGE_IMAGE, MAX_BINDLESS_RESOURCES }
};

VkDescriptorPoolCreateInfo poolInfo{};
poolInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_POOL_CREATE_INFO;
poolInfo.flags = VK_DESCRIPTOR_POOL_CREATE_UPDATE_AFTER_BIND_BIT_EXT;
poolInfo.maxSets = MAX_BINDLESS_RESOURCES * ArraySize(poolSizeBindless);
poolInfo.pPoolSizes = poolSizesBindless;
vkCreateDescriptorPool(vulkanDevice, &poolInfo, vulkanAllocationCallbacks, &vulkanDescriptorPool);
```

If you are already familiar with how to [[Creating a descriptor pool|create a descriptor pool]] the only difference here is that we are setting the **VK_DESCRIPTOR_POOL_CREATE_UPDATE_AFTER_BIND_BIT_EXT** this allows us to update descriptor sets after they have been bound.

The other thing to note here is the **MAX_BINDLESS_RESOURCES** which is the total number of textures that's going to live in that array. I currently have this set to 256 but I believe because this is an SSBO. To quote the documentation on this `The spec guarantees that SSBOs can be up to 128MB. Most implementations will let you allocate a size up to the limit of GPU memory.` So unless the textures are extremely huge we don't have to wrorry.
# References
##### Main Notes
[[Creating a bindless descriptor set layout]]
[[Enabling bindless textures]]
[[Creating a descriptor set layout]]
#### Source Notes
[[Bindless - Mastering Graphics Programming with Vulkan]]