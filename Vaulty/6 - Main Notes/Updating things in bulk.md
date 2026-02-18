2026-02-11 10:35
Status: #baby 
Tags: [[performance]] [[DOD]]
# Updating things in bulk

When you're wanting to update something if possible try to update them in one go rather than individually if possible. The best way to do this for real time systems is to use a queue. The steps would go
1) If possible instead of updating things now add them to an update queue.
2) Then update them all in bulk at the end of the frame, clearing the queue in the process.
3) Repeat.

The reason you would want to do this is that updating things in mass at the end of a frame is that all the memory is close together meaning you're more likely to get cache locality and fetch things from RAM more efficently.

Here are some code snippets of how we are updating textures in vulkan.
```c++
struct ResourceUpdate 
{
    ResourceUpdateType type;
    uint32_t handle;
    uint32_t currentFrame;
    uint32_t deleting;
};
Array<ResourceUpdate> textureToUpdateBindless;

void vulkanCreateTexture(GPUDevice& gpu)
{
    ResourceUpdate resourceUpdate{};
    //Build up the data for an element to go into the queue.

    gpu.textureToUpdateBindless.push(resourceUpdate);
}

void GPUDevice::present()
{
	//Frame stuff happens here...
    if (textureToUpdateBindless.size)
    {
        uint32_t currentWriteIndex = 0;
        for (int32_t it = textureToUpdateBindless.size - 1; it >= 0; --it)
        {
			//Update textures and add deleted textures to the delete queue.

            textureToUpdateBindless.deleteSwap(it);
        }
        
        //Add the updated textures to a descriptor set.
    }
}

```

1) We are have an  `Array<ResourceUpdate>` acting as a queue. 
2) We add the textures to a queue on creation in the function **vulkanCreateTexture**
3) We do something with the textures in the **present** function. While doing a delete swap on textures we have dealt with.

You can only do any of this if you seperate the data from the functions. Allow arrays to formed and things like without coupling it all together.
# References
##### Main Notes
[[What is data oriented design]]
#### Source Notes
[[Intro to Data Oriented Design for Games]]