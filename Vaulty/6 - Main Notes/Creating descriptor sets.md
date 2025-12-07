2025-11-21 19:08
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]] 
# Creating descriptor sets

If you've set up the descriptor pool you can now set up the descriptor sets. You need to allocate memory to them like this

```c++
std::vector<VkDescriptorSetLayout> layouts(imageCount, descriptorSetLayout);
VkDescriptorSetAllocateInfo descriptorAllocateInfo{};
descriptorAllocateInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_ALLOCATE_INFO;
descriptorAllocateInfo.descriptorPool = descriptorPool;
descriptorAllocateInfo.descriptorSetCount = imageCount;
descriptorAllocateInfo.pSetLayouts = layouts.data();
```

You need a list of descriptor set layouts for the amount of descriptor sets you want. That's what **.descriptorSetCount** and **.pSetLayout**. We've just used a little C++ magic to allocate the same object **descriptorSetLayout** for the amount of **imageCount**, with a copy. 

Finally we just give the descriptor pool **.descriptorPool**.

```c++
std::vector<VkDescriptorSet> descriptorSets(imageCount);
if (vkAllocateDescriptorSets(device, &descriptorAllocateInfo, descriptorSets.data()) != VK_SUCCESS) 
{
    printf("Failed to allocate descriptors sets");
}
```

Then we can allocate a set of descriptor sets. The function **vkAllocateDescriptorSets** expect there to be a identical set of descriptor set layouts in the **VkDescriptorSetAllocateInfo** struct which is why we had to copy/move them in our std::vector allocation.

We don't have to destroy these descriptor sets they will be cleaned up with the descriptor pool.

Now we have the descriptor sets they need to be configured and populated. 

```c++
for (uint32_t i = 0; i < imageCount; ++i) 
{
    VkDescriptorBufferInfo desBufferInfo{};
    desBufferInfo.buffer = uniformBuffers[i];
    desBufferInfo.offset = 0;
    desBufferInfo.range = sizeof(ModelData);
    
    //Other allocation stuff
}
```

The **VkDescriptorBufferInfo** specifies the region within the data for our descriptor to use and the buffer itself.  You can instead of using sizeof() use **VK_WHOLE_SIZE**. 

You update the descriptor set with **vkUpdateDescriptorSets** using the configuration struct **VkWriteDescriptorSet** 

```c++
for (uint32_t i = 0; i < imageCount; ++i) 
{
	//Buffer creation...

    VkWriteDescriptorSet descriptorWrite{};
    descriptorWrite.sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
    descriptorWrite.dstSet = descriptorSets[i];
    descriptorWrite.dstBinding = 0;
    descriptorWrite.dstArrayElement = 0;
    descriptorWrite.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
    descriptorWrite.descriptorCount = 1;
    descriptorWrite.pBufferInfo = &desBufferInfo;
    
    //Updated descriptor set...
}
```

The parameters **.dstSet** and **.dstBinding** specify the descriptor set you want to update and the binding. Descriptor sets can be arrays we so **.dstArrayElement** tells vulkan what is the first index of that array, if you're not using arrays set the to 0. 

**.descriptorType** specify the descriptor type again. Descriptor sets to update arrays of descriptor sets at the same time from **.dstArrayElement** to **.descriptorCount** If you don't have an array just set **.descriptorCount** to 1. 

```c++
descriptorWrite.pBufferInfo = &desBufferInfo;
descriptorWrite.pImageInfo = nullptr;
descriptorWrite.pTexelBufferView = nullptr;
```

Then you add what buffer type/ image you want. 
- **.pBufferInfo** is for buffers
- **.pImageInfo** is for image data
- **.pTexelBufferView** is for a buffer view.

We are using a regular buffer so **.pBufferInfo** it is.

```c++
vkUpdateDescriptorSets(device, 1, &descriptorWrite, 0, nullptr);
```

Then run the update function to update your descriptors, remember this can handle arrays so we pass in the reference with a size of 1. We are passing in the **VkWriteDescriptorSet** which writes data, you can also pass in **VkCopyDescriptorSet** this copies descriptor sets to another set.
# References
##### Main Notes
[[Creating a descriptor pool]]
#### Source Notes
[[Uniform buffers]]