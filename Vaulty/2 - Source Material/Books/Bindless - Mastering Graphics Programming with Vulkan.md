# Reference Mastering Graphics Programming with Vulkan

When you are using bindless you are still binding but you bind an list of resources that can be indexed into in the shaders. Basically every single desktop GPU support this, so use it. It makes using Vulkan easier to use. These resources will be mainly used for textures which all can be used across shaders. 
##### Checking bindless is supported
First thing you need to is make sure that **VK_EXT_descriptor_indexing** is supported. This is a device level so you need to query it at the device leve.

```c++
VkPhysicalDeviceDescriptorIndexingFeatures indexingFeatures{};
indexingFeatures.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_DESCRIPTOR_INDEXING_FEATURES;

VkPhysicalDeviceFeatures2 deviceFeatures{};
deviceFeatures.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_FEATURES_2;
deviceFeatures.pNext = &indexingFeatures;

vkGetPhysicalDeviceFeatures2(vulkanPhysicalDevice, &deviceFeatures);

bindlessSupported = indexingFeatures.descriptorBindingPartiallyBound && indexingFeatures.runtimeDescriptorArray;
```

This is a little odd you need to have both **VkPhysicalDeviceDescriptorIndexingFeatures** and **VkPhysicalDeviceFeatures2** chained together and queried. The reason behind that is that the bindless is actually 2 features`descriptorBindingPartiallyBound` from the **VkPhysicalDeviceDescriptorIndexingFeatures** and `runtimeDescriptorArray` from the  **VkPhysicalDeviceFeatures2** struct.

If `bindlessSupported` is true we can now enable it.

```c++
VkPhysicalDeviceFeatures2 physicalDeviceFeature2{};
physicalDeviceFeature2.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_FEATURES_2;

VkPhysicalDeviceDescriptorIndexingFeatures indexingFeatures{};
indexingFeatures.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_DESCRIPTOR_INDEXING_FEATURES;

void* currentPnext;
if (bindlessSupported)
{
    indexingFeatures.pNext = currentPnext;
    currentPnext = &indexingFeatures;
}

physicalDeviceFeature2.pNext = currentPnext;
vkGetPhysicalDeviceFeatures2(vulkanPhysicalDevice, &physicalDeviceFeature2);
```

Using the **VkPhysicalDeviceDescriptorIndexingFeatures** struct in the linked list of dependencies you build up when turning on features.
##### Creating the descriptor pools.
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
##### Bindless descriptor set layout
After we have the descriptor pool we now need to create a descriptor set layout that allows for bindless descriptor sets. 

Now, for your bindless textures you'll want to match the descriptor pool we created with the **VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER** and **VK_DESCRIPTOR_TYPE_STORAGE_IMAGE** as things that are part of the descriptor set layout.

I create the descriptor set layout at the end of the init() function for the GPUDevice
```c++
DescriptorSetLayoutCreation bindlessLayoutCreation{};
bindlessLayoutCreation
	.addBinding({ VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER, BINDLESS_TEXTURE_BINDING, MAX_BINDLESS_RESOURCES, VK_SHADER_STAGE_ALL, "BindlessTextures" })
	.addBinding({ VK_DESCRIPTOR_TYPE_STORAGE_IMAGE, BINDLESS_IMAGE_BINDING, MAX_BINDLESS_RESOURCES, VK_SHADER_STAGE_ALL, "BindlessImages" })
	.setSetIndex(0)
	.setName("BindlessLayout");
bindlessLayoutCreation.bindless = true;

bindlessDescriptorSetLayoutHandle = createDescriptorSetLayout(bindlessLayoutCreation);
```

Notice in the third argument we have **MAX_BINDLESS_RESOURCES** instead of one, this is to accomodate how many textures we can use per draw call. 

Next we get the vulkan descriptor set to use later in the code base.

```c++
DescriptorSetCreation bindlessSetCreation;
bindlessSetCreation.reset()
    .setLayout(bindlessDescriptorSetLayoutHandle);
bindlessDescriptorSetHandle = createDescriptorSet(bindlessSetCreation);

DescriptorSet* bindlessSet = accessDescriptorSet(bindlessDescriptorSetHandle);
bindlessDescriptorSet = bindlessSet->vkDescriptorSet;
```

Here we are only interesting in the line `bindlessDescriptorSet = bindlessSet->vkDescriptorSet;` that's what the rest of the code is doing.
##### Updating bindless textures 
Now this is a little interesting, because with bindless we can update thing after they are bound now we have a decision to make **WHEN** do you update textures with a **vkUpdateDescriptorSets**. Well if you're interested in data oriented design we have a optimisation you can make to update thing in bulk with [[Updating things in bulk]]. The idea here is we build a queue of textures that need to be updating and once they have been dealt with we remove them from the queue.


```c++
//Load textures in texture here ^^
if (gpu.bindlessSupported)
{
    ResourceUpdate resourceUpdate{};
    resourceUpdate.type = ResourceUpdateType::TEXTURE;
    resourceUpdate.handle = texture->handle.index;
    resourceUpdate.currentFrame = gpu.currentFrame;
    resourceUpdate.deleting = 0;

    gpu.textureToUpdateBindless.push(resourceUpdate);
}
```

The important part here is that we take the texture handle. 

We now have to loop through all the `textureToUpdateBindless` elements slowly writing them into our descriptor set, while removing that texture from the queue. Remember at this point for bindless textures we should already handles to textures that has a **VkImage** and a **VkImageView**.

Please review [[Creating descriptor sets]] and [[Creating an combined image sampler descriptor]] as what we are doing here is efficiently the same thing but we are updating multiple descriptor sets with image samplers at the same time. 

The comments will explain what this code does.
```c++
//We only run this if there is something in the queue. It's very likely on the second run through aka the second frame the queue will be empty.
if (textureToUpdateBindless.size)
{
    //Here we have an array of VkWriteDescriptorSet which will be loaded with relivent data as we loop through the result.
    VkWriteDescriptorSet bindlessDescriptorWrites[MAX_BINDLESS_RESOURCES];
    //Matching that we have the VkDescriptorImageInfo which will write the VkImageView into the descriptor set.
    VkDescriptorImageInfo bindlessImageInfo[MAX_BINDLESS_RESOURCES];

	//We need a dummy texture to write a texture to our descriptor that has nothing in it, this allows us to delete textures without having to recompile the engine or change the descriptor sets amount.
    Texture* vkDummyTexture = accessTexture(dummyTexture);

	//This is here to keep track of how many descriptor sets we are updating.
    uint32_t currentWriteIndex = 0;
    //We are going negatively through this array because we are deleting elements from the back of the array as we go.
    for (int32_t it = textureToUpdateBindless.size - 1; it >= 0; --it)
    {
	    //We are using a non-const reference to update the elements in this array.
        ResourceUpdate& textureToUpdate = textureToUpdateBindless[it];
        //Here we have the texture we have already loaded with it's VKImage and VkImageView.
        Texture* texture = accessTexture({ textureToUpdate.handle });

		//If this texture hasn't been loaded we skip it.
        if (texture->vkImageView == VK_NULL_HANDLE)
        {
            continue;
        }

		//Next we build up the data to write to.
		//Again Non-const reference for the bindlessDescriptorWrites[MAX_BINDLESS_RESOURCES];
        VkWriteDescriptorSet& descriptorWrite = bindlessDescriptorWrites[currentWriteIndex];
        descriptorWrite = { VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET };
        descriptorWrite.descriptorCount = 1;
        descriptorWrite.dstArrayElement = textureToUpdate.handle;
        //You'll see later that we add a default sampler even if there isn't a sampler for that texture, this is why vvvv
        descriptorWrite.descriptorType = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
        descriptorWrite.dstSet = bindlessDescriptorSet;
        //BINDLESS_TEXTURE_BINDING == 10 because that's the bindless texture location of the things will be indexing through.
        descriptorWrite.dstBinding = BINDLESS_TEXTURE_BINDING;

        //Handles should be the same.
        VOID_ASSERT(texture->handle.index == textureToUpdate.handle);

		//We need to get a default sampler because it's not always the case that we would want an image sampler combined. So if we want just an image then we want to update our image with a default empty sampler.
        Sampler* vkDefaultSampler = accessSampler(defaultSampler);
        //Here we need the non-const reference to bindlessImageInfo array.
        VkDescriptorImageInfo& descriptorImageInfo = bindlessImageInfo[currentWriteIndex];

		//If we are deleting a texture we don't update the VkDescriptorImageInfo with the textures in the queue. If we aren't deleting the we update the VkDescriptorImageInfo with the VkImageView and VkSampler in the queue.
        if (!textureToUpdate.deleting)
        {
            //Here we set the image view.
            descriptorImageInfo.imageView = texture->vkImageView;
            if (texture->sampler != nullptr)
            {
	            //If we have a sampler, we add it.
                descriptorImageInfo.sampler = texture->sampler->vkSampler;
            }
            else
            {
	            //If we don't have sampler we set the default one.
                descriptorImageInfo.sampler = vkDefaultSampler->vkSampler;
            }
        }
        else
        {
            //If we are deleting a texture we set the image view and the sampler to the defaults aka an empty non-texture.
            descriptorImageInfo.imageView = vkDummyTexture->vkImageView;
            descriptorImageInfo.sampler = vkDefaultSampler->vkSampler;
        }

		//Since this descriptor is going to be accessed the in the shader the layout should be VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL
        descriptorImageInfo.imageLayout = VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL;
        //Once we have set the data to upload to the descriptor we write this to the shader.
        descriptorWrite.pImageInfo = &descriptorImageInfo;
		
		//Next we delete the texture from the queue.
        textureToUpdate.currentFrame = UINT32_MAX;
        textureToUpdateBindless.deleteSwap(it);
        
		//Everytime we have created the descriptorWrite and descriptorImageInfo we keep track of it.
        ++currentWriteIndex;

        //Add texture to the delete queue if we are deleting the texture to be deleted in bulk later.
        if (addTextureToDelete)
        {
            resourceDeletionQueue.push(
                { 
	                .type = ResourceUpdateType::TEXTURE,
	                .handle = texture->handle.index, 
                    .currentFrame = currentFrame, 
                    .deleting = 1 
                });
        }
    }

	//Outside of the for loop we write the data to the vkUpdateDescriptorSets in the array of the bindlessDescriptorWrites with the total currentWriteIndex.
    if (currentWriteIndex)
    {
        vkUpdateDescriptorSets(vulkanDevice, currentWriteIndex, bindlessDescriptorWrites, 0, nullptr);
    }
}
```