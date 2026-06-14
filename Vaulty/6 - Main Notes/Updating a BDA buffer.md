2026-05-27 08:38
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]] [[buffer device address]]
# Updating a BDA buffer

If you have a standard vkBuffer that is accessed via BDA and you're updating the data on the CPU. This needs to be host visible and conherent.
#### IMPORTANT
This is true for any type of vkBuffer. If it's CPU visible and conherent buffers these need to be updated per-frames in flight.

When you update the data within a BDA and it's CPU visible and conherent, you simple update using the like this.

```c++
struct Data
{
	uint32_t something;
}

//Assuming here we have 2 frames in flight.
Array<Data> dataArray[2];

//Create the bindless buffer here [[Creating a BDA buffer]] for more info on that.

while(gpu.running)
{
	//Events here
	//Physics here
	
	Buffer* buffer = gpu.accessBuffer(handles[gpu.frameInFlight]);
	
	for(uint32_t i = 0; i < dataArray[gpu.frameInFlight].size; ++i)
	{
		dataArray[i].something = cglms_vec3_dot(vec1, vec2);
	}
	
	if (dataArray[gpu.frameInFlight].data)
	{
		vmaCopyMemoryToAllocation(gpu.VMAAllocator, dataArray[gpu.frameInFlight].data, buffer->vmaAllocation, 0, sizeof(Data) * dataArray[gpu.frameInFlight].size);
	}
}
```

There are two things to be aware of here. The `vmaCopyMemoryToAllocation()` need the destination pointer found in the second argument to not be null.

The most **IMPORTANT** thing here though is that you reuse the **SAME** pointer that points to the object you used to create the bindless buffer. You **CANNOT** make a new piece of memory and have that uploaded to the GPU, that corrupts memory in strange ways.

Also note because this is a CPU updated buffer, you need to update things using the current frames in flight to make sure you don't have a race condition with your buffers.
# References
##### Main Notes
[[What is Buffer Device Address (BDA)]]
[[Creating a BDA buffer]]
#### Source Notes
