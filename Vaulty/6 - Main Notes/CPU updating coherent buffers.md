2026-05-27 08:22
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# CPU updating coherent buffers

If you have a buffer that needs to be updated on the CPU and this touches the GPU. You need to be careful, even if you using VMA with a `vmaCopyMemoryToAllocation` which creating the mapping between the CPU and GPU, you could end up in a race condition.

The problem here is frame in flight. If you have a CPU coherent and visible buffer and you update in on frame N, and you have using only 1 buffer, might overwrite memory that the GPU is already reading from.

What you would need to do in this sitation is make 2 buffers for each frame in flight.

```c++
#include "Foundation/Memory.hpp"
#include "Graphics/GPUDevice.hpp"

struct Data
{
	uint32_t something;
};

int main()
{
	GPUDevice gpu;
	gpu.init();

	//We are assuming that we never change our frames in flight.
	BufferHandle handles[2];
	Array<Data> dataArray;
	dataArray.init(&MemoryService::instance()->systemAllocator, gpu.framesInFlight, gpu.framesInFlight);
	
	dataArray.({1});
	dataArray.({2});
	dataArray.({3});
	
	for()
	{
		BufferCreation bufferCreation{};
		bufferCreation.set(VK_BUFFER_USAGE_UNIFORM_BUFFER_BIT, sizeof(Data) * dataArray.size)
	    .setName("debugRenderer")
	    .setData(dataArray.data);
	    
	    handles[i] = gpu.createBuffer(bufferCreation);
    }
    
	while(gpu.running)
	{
		//Events here
		//Physics here
		
		Buffer* buffer = gpu.accessBuffer(handles[gpu.frameInFlight]);
		
		if (dataArray.data)
        {
	        vmaCopyMemoryToAllocation(gpu.VMAAllocator, dataArray.data, buffer->vmaAllocation, 0, sizeof(Data) * dataArray.size);
        }
	}

	return 0;
}
```

Now you have a safely updated frame in flight buffer. It's important to note here though that in this example that the buffer is going to be updated at run time constantly anyway. This will be a little tricker if you only need to update this buffer after an event. You'll need to update second buffer that's currently being used by the GPU in the next frame. If you don't do this you'll get flashing.
# References
##### Main Notes
[[Creating a BDA buffer]]
#### Source Notes
