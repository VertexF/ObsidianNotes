2025-12-01 08:54
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]] [[vulkan descriptors]]
# Creating an combined image sampler descriptor

To allow us to access an image view that contains a texture in a shader we need to be able to use a descriptor set type called a **combined image sampler**. 

The first thing you have to do is create a sampler descriptor with **VkDescriptorSetLayoutBinding** and add it to the descriptor then add that to the **DescriptorSetLayoutCreateInfo** This is the same as [[Creating a descriptor set layout]] but we make it a combined image sampler.

```c++
VkDescriptorSetLayoutBinding sampleLayoutBinding{};
sampleLayoutBinding.binding = 1;
sampleLayoutBinding.descriptorCount = 1;
sampleLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
sampleLayoutBinding.pImmutableSamplers = nullptr;
sampleLayoutBinding.stageFlags = VK_SHADER_STAGE_FRAGMENT_BIT;
    
//Pipeline layout creation
std::array<VkDescriptorSetLayoutBinding, 2> binding = { uboLayoutBinding, sampleLayoutBinding };
VkDescriptorSetLayoutCreateInfo layoutInfo{};
layoutInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_LAYOUT_CREATE_INFO;
layoutInfo.bindingCount = uint32_t(binding.size());
layoutInfo.pBindings = binding.data();
```

Here we are creating the descriptor but this time it's a **VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER** type. You can put image sampler in the vertex shader if you need to generate a height map, by deforming a vertex grid.

We also need to increase the size of the descriptor pool to include the sampler
```c++
//Descriptor pool creation
std::array<VkDescriptorPoolSize, 2> poolSize{};
poolSize[0].type = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
poolSize[0].descriptorCount = imageCount;
poolSize[1].type = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
poolSize[1].descriptorCount = imageCount;

VkDescriptorPoolCreateInfo poolDescriptorCreateInfo{};
poolDescriptorCreateInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_POOL_CREATE_INFO;
poolDescriptorCreateInfo.poolSizeCount = uint32_t(poolSize.size());
poolDescriptorCreateInfo.pPoolSizes = poolSize.data();
poolDescriptorCreateInfo.maxSets = imageCount;
```

Since the allocation of the descriptors and left to the graphics driver you don't need to set things up in away that types are exact, but it's best practice to do so.

Next we'll need to write the the image sampler descriptor set to be able to bind this at draw time.
```c++
VkDescriptorBufferInfo desBufferInfo{};
desBufferInfo.buffer = uniformBuffers[i];
desBufferInfo.offset = 0;
desBufferInfo.range = sizeof(ModelData);

VkDescriptorImageInfo desImageInfo{};
desImageInfo.imageLayout = VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL;
desImageInfo.imageView = textureImageView;
desImageInfo.sampler = textureSampler;

std::array<VkWriteDescriptorSet, 2> descriptorWrites{};

descriptorWrites[0].sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
descriptorWrites[0].dstSet = descriptorSets[i];
descriptorWrites[0].dstBinding = 0;
descriptorWrites[0].dstArrayElement = 0;
descriptorWrites[0].descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
descriptorWrites[0].descriptorCount = 1;
descriptorWrites[0].pBufferInfo = &desBufferInfo;
descriptorWrites[0].pImageInfo = nullptr;
descriptorWrites[0].pTexelBufferView = nullptr;

descriptorWrites[1].sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
descriptorWrites[1].dstSet = descriptorSets[i];
descriptorWrites[1].dstBinding = 1;
descriptorWrites[1].dstArrayElement = 0;
descriptorWrites[1].descriptorType = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
descriptorWrites[1].descriptorCount = 1;
descriptorWrites[1].pBufferInfo = nullptr;
descriptorWrites[1].pImageInfo = &desImageInfo;
descriptorWrites[1].pTexelBufferView = nullptr;

vkUpdateDescriptorSets(device, uint32_t(descriptorWrites.size()), descriptorWrites.data(), 0, nullptr);
```

Note we are updating the **.pImageInfo** with out image data with unlike the **VkDescriptorBufferInfo** struct.
# References
##### Main Notes
[[Creating a descriptor pool]]
[[Creating a descriptor set layout]]
[[Binding a descriptor set to use at draw time]]
[[Creating descriptor sets]]
#### Source Notes
[[Texture mapping]]