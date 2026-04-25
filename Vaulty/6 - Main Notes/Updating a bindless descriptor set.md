2026-02-27 09:17
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]] [[vulkan textures]]
# Updating a bindless descriptor set

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

		//If we are deleting a texture we don't update the VkDescriptorImageInfo with the textures in the queue. 
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

		//Since this descriptor is going to be accessed the in the shader the layout should be VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL
        descriptorImageInfo.imageLayout = VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL;
        //Once we have set the data to upload to the descriptor we write this to the shader.
        descriptorWrite.pImageInfo = &descriptorImageInfo;
		
		//Next we delete the texture from the queue.
        textureToUpdate.currentFrame = UINT32_MAX;
        textureToUpdateBindless.deleteSwap(it);
        
		//Everytime we have created the descriptorWrite and descriptorImageInfo we keep track of it.
        ++currentWriteIndex;
    }

	//Outside of the for loop we write the data to the vkUpdateDescriptorSets in the array of the bindlessDescriptorWrites with the total currentWriteIndex.
    if (currentWriteIndex)
    {
        vkUpdateDescriptorSets(vulkanDevice, currentWriteIndex, bindlessDescriptorWrites, 0, nullptr);
    }
}
```

Why are we doing this? Well we have array of textures that we are going to index into, so we need to update them when we are going to use them. This safe to do **IF** you do this before a draw call. It's also important to note we are doing update after bind here. We bind all the textures and then update them afterwards.
# References
##### Main Notes
[[Updating things in bulk]]
[[Creating a bindless descriptor set layout]]
[[Creating descriptor sets]] 
[[Creating the bindless descriptor pool]]
[[Creating an combined image sampler descriptor]]
#### Source Notes
[[Bindless - Mastering Graphics Programming with Vulkan]]