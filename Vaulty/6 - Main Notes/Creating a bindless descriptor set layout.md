2026-02-27 08:54
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]] [[vulkan descriptors]]
# Creating a bindless descriptor set layout

After we have the [[Creating the bindless descriptor pool|descriptor pool]] we now need to create a descriptor set layout that allows for bindless descriptor sets. 

Now, for your bindless textures you'll want to match the descriptor pool we created with the **VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER** and **VK_DESCRIPTOR_TYPE_STORAGE_IMAGE** as things that are part of the descriptor set layout.

I create the descriptor set layout at the end of the init() function for the GPUDevice
```c++
uint32_t MAX_BINDLESS_RESOURCES = 256;

//Here we have an image + sampler binding
Binding textureBinding{};
textureBinding.type = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
textureBinding.index = 0;
textureBinding.size = MAX_BINDLESS_RESOURCES;
textureBinding.shaderStage = VK_SHADER_STAGE_ALL;

//Here we have just an image binding.
Binding imageBinding{};
imageBinding.type = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
imageBinding.index = 1;
imageBinding.size = MAX_BINDLESS_RESOURCES;
imageBinding.shaderStage = VK_SHADER_STAGE_ALL;

DescriptorSetLayoutCreation bindlessLayoutCreation{};
bindlessLayoutCreation
	.addBinding(textureBinding)
	.addBinding(imageBinding)
	.setSetIndex(1)
bindlessLayoutCreation.bindless = true;

bindlessDescriptorSetLayoutHandle = createDescriptorSetLayout(bindlessLayoutCreation);
```

Notice that **.size** has the **MAX_BINDLESS_RESOURCES** instead of one, this is to accomodate how many textures we can use per draw call. This can be much bigger than 256 if you need more textures in a scene than that.

Next we get the descriptor set that was created as part of this descriptor set layout.

```c++
DescriptorSet* bindlessSet = accessDescriptorSet(bindlessDescriptorSetHandle);
bindlessDescriptorSet = bindlessSet->vkDescriptorSet;
```

Here we are only interesting in the line `bindlessDescriptorSet = bindlessSet->vkDescriptorSet;` that's what the rest of the code is doing.
# References
##### Main Notes
[[Creating a descriptor set layout]]
[[Creating the bindless descriptor pool]]
[[Updating a bindless descriptor set]]
#### Source Notes
[[Bindless - Mastering Graphics Programming with Vulkan]]