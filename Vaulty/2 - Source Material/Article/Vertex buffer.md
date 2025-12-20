# Reference https://vulkan-tutorial.com/Vertex_buffers/Vertex_input_description

##### Vertex input description
If you want to use a shader that gets data uploaded to the GPU to be processed. To process shader vertex attributes that take in vertex data as input in the shader and do something you with them you need to create inputs for the vertex shader. The simplest set up would look something like this

```c++
layout (location = 0) in vec2 inPosition;
layout (location = 1) in vec3 inColour; 

layout(location = 0) out vec3 fragColour;

void main()
{
	gl_Position = vec4(inPosition, 0.0, 1.0);
	fragColour = inColour;
}
```

This takes in both position and colour as inputs and uses them in the shader. Remember this shader runs per vertex.

Not all types take up simply 1 slot in the shader. The x64 version of vec3 called dvec3 actually take up 2 slots. If you need something like this look at the OpenGL layout qualifers https://wikis.khronos.org/opengl/Layout_Qualifier_(GLSL).
##### Vertex data
There isn't much to be said here about vertex data. This could be just position or other stuff. For this tutorial we are creating colour and position with a Vertex struct that contains these data types.

This will eventually come from models so this isn't important really.
##### Binding descriptions

The first to do once you've set up the data is create vertex binding descriptions. 
```c++
static VkVertexInputBindingDescription getBindingDescriptions() 
{
    VkVertexInputBindingDescription bindingDescription{};
    bindingDescription.binding = 0;
    bindingDescription.stride = sizeof(Vertex);
    bindingDescription.inputRate = VK_VERTEX_INPUT_RATE_VERTEX;
    return bindingDescription;
}
```

First we need to say what the where to bind the data in the vertex shader which is the **.binding**. Next we need to set the stride which is the **.stride** and finally we need to set the rate on which we will send in the data this is **.inputRate**.

The **.inputRate** can be either these two values
- **VK_VERTEX_INPUT_RATE_VERTEX** Move to the next entry after each vertex.
- **VK_VERTEX_INPUT_RATE_INSTANCE** Move to the next data entry after each instance. 

This will change depending on what your vertex data is layed out like and how you're rendering.
##### Attribute descriptions

Next you want to describe the vertex descriptions that are in the buffer.

```c++
static std::array<VkVertexInputAttributeDescription, 2> getAttributeDescriptions() 
{
    std::array<VkVertexInputAttributeDescription, 2> attributeDescriptions{};

    attributeDescriptions[0].binding = 0;
    attributeDescriptions[0].location = 0;
    attributeDescriptions[0].format = VK_FORMAT_R32G32_SFLOAT;
    attributeDescriptions[0].offset = offsetof(Vertex, position);

    attributeDescriptions[1].binding = 0;
    attributeDescriptions[1].location = 1;
    attributeDescriptions[1].format = VK_FORMAT_R32G32B32_SFLOAT;
    attributeDescriptions[1].offset = offsetof(Vertex, colour);

    return attributeDescriptions;
}
```

So we are still at binding 0 but each location needs to be set up. Every location you see in the vertex shader needs to have the right **.location** lining it up. For example `layout (location = 0) in vec2 inPosition;` is `.location = 0;`

Then we have offsetof for the stride per vertex attribute and the **.format** which should be these values to match these types in the shader:

- float = VK_FORMAT_R32_SFLOAT
- vec2 = VK_FORMAT_R32G32_SFLOAT
- vec3 = VK_FORMAT_R32G32B32_SFLOAT
- vec4 = VK_FORMAT_R32G32B32A32_SFLOAT
- ivec2 = VK_FORMAT_R32G32_SINT
- uvec2 = VK_FORMAT_R32G32B32A32_SUINT
- double = VK_FORMAT_R64_SFLOAT

If you get the format wrong the **.format** effects how the offsetof will be handled. So you'll also mess that wrong. It also just silently fails so be careful.
##### Pipeline vertex input

Now you just add these to the pipeline like any other part of the graphics pipeline and the vertex inputs should be ready to go.

```c++
VkPipelineVertexInputStateCreateInfo vertexInputState{};
vertexInputState.sType = VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO;
vertexInputState.vertexBindingDescriptionCount = 1;
vertexInputState.pVertexBindingDescriptions = &bindingDescription;
vertexInputState.vertexAttributeDescriptionCount = uint32_t(attributeDescriptions.size());
vertexInputState.pVertexAttributeDescriptions = attributeDescriptions.data();
```

##### Vertex buffer intro
These can store anything that's read by the GPU. They also don't allocate there own memory, meaning you'll have to do extra work to load in the data needed.
##### Vertex buffer creation

Lets begin
```c++
VkBufferCreateInfo bufferInfo{};
bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
bufferInfo.size = sizeof(vertices[0]) * vertices.size();
bufferInfo.usage = VK_BUFFER_USAGE_VERTEX_BUFFER_BIT;
bufferInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;
```

We have the **.size** which is the size of the buffer that's going to be used. **.usage** is the usage of the buffer, you can OR more usage types together if you want. Finally we have **.sharingMode** which like the swapchain images, you can specify if you're doing to use them with different queue families or not. We aren't going to use this any where other than the graphics queue.

```c++
VkBuffer vertexBuffer;
if (vkCreateBuffer(device, &bufferInfo, nullptr, &vertexBuffer) != VK_SUCCESS) 
{
    printf("Failed to create vertex buffer.");
}
```

Then you create the buffer. Deleting is very similar

```c++
vkDestroyBuffer(device, vertexBuffer, nullptr);
```
##### Memory requirements
Now we need to allocate the memory into the buffer. To begin with we need to buffer type already created to get memory requirements needed to allocate data into the buffer.

```c++
VkMemoryRequirements memoryRequirements;
vkGetBufferMemoryRequirements(device, vertexBuffer, &memoryRequirements);
```

Memory requirements for buffer are returned in this struct:
- **.size** which is the size of memory in bytes.
- **.memoryTypeBits** which is a bit mask that allows us to see if various bit types are supported. 
- **.aligment** which tells if and were the offset of buffer begins in the allocated region. This changes depending on the **VkBufferCreateInfo** **.usage** and **.flags**.

The graphics card can do a lot of different allocation, so we need to figure out which we need to use for our case.
```c++
VkMemoryAllocateInfo bufferAllocationInfo{};
bufferAllocationInfo.sType = VK_STRUCTURE_TYPE_MEMORY_ALLOCATE_INFO;
bufferAllocationInfo.allocationSize = memoryRequirements.size;

uint32_t memoryType = UINT64_MAX;
uint32_t typeFilter = memoryRequirements.memoryTypeBits;
VkMemoryPropertyFlags properties = VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT;

VkPhysicalDeviceMemoryProperties memoryProperties;
vkGetPhysicalDeviceMemoryProperties(physicalDevice, &memoryProperties);
```

Using this function **vkGetPhysicalDeviceMemoryProperties** we get the properties for what type of heaps we can use. Within the **VkPhysicalDeviceMemoryProperties** struct you have 2 arrays.
```c++
    VkMemoryType    memoryTypes[VK_MAX_MEMORY_TYPES];
    VkMemoryHeap    memoryHeaps[VK_MAX_MEMORY_HEAPS];
```

These two type of both of these select if we are using the VRAM on a graphics card or the RAM swap space on the hosts side. There are different heaps when allocating buffers. You can ignore which heap you use BUT this can affect performance. You **HAVE** to select the memory type which is what we will do here.

```c++
uint32_t memoryType = UINT32_MAX;
for (uint32_t i = 0; i < memoryProperties.memoryTypeCount; ++i) 
{
    if ((typeFilter & (1 << i)) && (memoryProperties.memoryTypes[i].propertyFlags & properties) == properties)
    {
         memoryType = i;
    }
}

if (memoryType == UINT64_MAX) 
{
    printf("Failed to find suitable memory type.");
}
```

There are 2 check here the
1) `(typeFilter & (1 << i))` **typeFilter** is a bit mask that contains 1's in binary. Each 1 tells us that a type of allocation is value. So we step through each bit checking it's with the `& (1 << i)` part.
2) Next we need to check again the appropriated memory type looping through the **memoryTypes** and comparing against the **VkMemoryPropertyFlags** we set up eariler. The reason we do an AND operation and then see if it's equal **properties** is because ANDing may leave the result as non-zero and NOT equal to the property we want.

If both things are valid now we have select the approcate allocator for our buffer.

```c++
VkDeviceMemory vertexBufferMemory;
if (vkAllocateMemory(device, &bufferAllocationInfo, nullptr, &vertexBufferMemory) != VK_SUCCESS)
{
    printf("Failed to allocate the vertex buffer object");
}

vkBindBufferMemory(device, vertexBuffer, vertexBufferMemory, 0);
```

Then you allocate, and bind we sets the vertex buffer and memory to be linked together. Right now there is nothing in the memory so you'll need to fill the buffer with something before you use it.

vkBindBufferMemory last arguments is the region in memory were the offset begins. It requires things to be divided by **.alignment** from the **VkPhysicalDeviceMemoryProperties** struct
##### Filling up a vertex buffer with CPU visibility

```c++
void* data;
vkMapMemory(device, vertexBufferMemory, 0, bufferInfo.size, 0, &data);
memcpy(data, vertices.data(), size_t(bufferInfo.size));
vkUnmapMemory(device, vertexBufferMemory);
```

When using one buffer to load memory into the buffer you need to be aware that there are two ways of doing something. 
1) Is using the memory heap that is host coherent with VK_MEMORY_PROPERTY_HOST_COHERNET_BIT
2) Flushing the buffer with **vkFlushMappedMemoryRanges** when you're writing to a buffer and call **vkInvalidateMappedMemoryRanges** before reading the mapped memory.

You need to worry about this because the GPU doesn't guarateen when it will be uploaded because of caching. Correctly we use VK_MEMORY_PROPERTY_HOST_COHERNET_BIT which is slower. The only thing we can be sure about is that it will be available when we get to the **vkQueueSubmit**.
##### Binding the vertex buffer
To use the vertex buffer you've gotta record the binding.

```c++
VkBuffer vertexBuffers[] = { vertexBuffer };
VkDeviceSize offsets[] = { 0 };
vkCmdBindVertexBuffers(commandBuffers[currentFrame], 0, 1, vertexBuffers, offsets);

vkCmdDraw(commandBuffers[currentFrame], uint32_t(vertices.size()), 1, 0, 0);
```

The second two paramters in the **vkCmdBindVertexBuffers** function are the firstBinding which is the index of the first vertex input binding and bindingCount which is how many vertex buffers you're binding.

We also need to change the **vkCmdDraw** to actually set the vertices size which is how many vertices we have.
##### What are staging buffers
Staging buffer are used as an intermediate buffer before the final buffer. They are used to upload data to and then we transfer the data to the final buffer. The reason you would want to do this with something like the vertex buffer is that host visible buffered used on the GPU aren't typically good for performance. 

So the idea goes you want a buffer on the GPU to have the **VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT** which means it's local to the GPU, meaning two things. 1) It's faster and 2) The host can't see the memory. So you use a staging buffer that is visible to the host upload that, and then do a transfer command with a command buffer to move it over to the buffer we care about.

After that we destroy and delete the staging buffer once it's down.
##### Creating a transfer queue
When you want to transfer data from one buffer to another you'll need to do that with a command on a command buffer. Meaning you'll need a queue that supports the transfer commands. You can just use the graphics queue as these support. I didn't I made this work on it's own transfer queue.

To be able to do this you need to follow these steps.
1) You need to look for a queue that isn't VK_QUEUE_GRAPHICS_BIT and just a VK_QUEUE_TRANSFER_BIT. This is done as part of the family index look up. So you should at least two different queue families indices.
2) You need to make another VkQueue which is part of the logical device creation with the queue family index.
3) Next you'll next need to make a seperate VkCommandPool that is created with the transfer queue families index. You want to create this ONCE like the other one.
4) Then you can create a command buffer that runs transfer commands outside of the "main" queue.

Note you don't have to make the buffers that are used in different queue **.sharingMode** VK_SHARING_MODE_CONCURRENT. 
##### Setting up a staging buffer

It's important to note that you likely want to abstract creation of buffers into functions if you're planning on doing this. I'm also going to create a vertex buffer with a staging buffer the same patterns apply.

```c++
VkDeviceSize bufferSize = sizeof(vertices[0]) * vertices.size();

VkBuffer stagingBuffer;
VkDeviceMemory stagingBufferMemory;

createBuffer(bufferSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT, 
VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, stagingBuffer, stagingBufferMemory);

void* data;
vkMapMemory(device, stagingBufferMemory, 0, bufferSize, 0, &data);
memcpy(data, vertices.data(), size_t(bufferSize));
vkUnmapMemory(device, stagingBufferMemory);
```

Okay so the **createBuffer** is a help function that creates a buffer, selects the memory type and binds it. You can find more about that in the [[Setting up the a buffer]] and [[Allocating data to a buffer]] The arguments for this function are:
1) **VkDeviceSize** which takes in the size of the buffer. 
2) **VkBufferUsageFlags** This tells the function what the buffer is going to be used for. 
3) **VkMemoryPropertyFlags** This tell the function the memory type we are intested in. 
4) **VkBuffer&** is the output variable for the buffer we are going to use. 
5) **VkDeviceMemory&** is the output variable for the buffer **memory** we are going to use. 

The **VkBufferUsageFlags** are important to make this buffer transferable. The two values you want to know about for this are.
- **VK_BUFFER_USAGE_TRANSFER_SRC_BIT** Usage for when buffer are the source of the memory transfer operation.
- **VK_BUFFER_USAGE_TRANSFER_DST_BIT** Usage for when buffer are the destination of the memory transfer operation.

Remember since the **VkMemoryPropertyFlags** is host visible and coherent we can upload the memory into the buffer no problem.
##### Copying buffer data over
You may want to turn this into a helper function. I have a followed the steps [[Creating a transfer queue for buffer copying]] which is different frm the tutorial code.

The first thing you need to do is create a command buffer

```c++
VkCommandBufferAllocateInfo allocateInfo{};
allocateInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO;
allocateInfo.level = VK_COMMAND_BUFFER_LEVEL_PRIMARY;
allocateInfo.commandPool = transferCommandPool;
allocateInfo.commandBufferCount = 1;

VkCommandBuffer commandBuffer;
vkAllocateCommandBuffers(device, &allocateInfo, &commandBuffer);
```

Next you'll want to record to it.

```c++
VkCommandBufferBeginInfo beginInfo{};
beginInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO;
beginInfo.flags = VK_COMMAND_BUFFER_USAGE_ONE_TIME_SUBMIT_BIT;

vkBeginCommandBuffer(commandBuffer, &beginInfo);

VkBufferCopy copyRegion{};
copyRegion.size = size;
vkCmdCopyBuffer(commandBuffer, srcBuffer, dstBuffer, 1, &copyRegion);

vkEndCommandBuffer(commandBuffer);
```

We want to begin the recording with **VK_COMMAND_BUFFER_USAGE_ONE_TIME_SUBMIT_BIT** as we are only gonna use this once. 

Next you want to record the **vkCmdCopyBuffer**. This needs to take a **srcBuffer** that has the TRANSFER_SRC_BIT as it's usage and the TRANSFER_DST_BIT for the usage for the other bit. For the destination you also need to OR the other usage flags together so we can actually use the buffer for something.

For the case of copying data into a vertex buffer you'll want to the usage like this:


The **VkBufferCopy** just sets up how much of the source buffer you're copying over to the destination.

Then you submit it
```c++
VkSubmitInfo submitInfo{};
submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;
submitInfo.commandBufferCount = 1;
submitInfo.pCommandBuffers = &commandBuffer;

vkQueueSubmit(transferQueue, 1, &submitInfo, VK_NULL_HANDLE);
vkQueueWaitIdle(transferQueue);
```

The faster way of waiting things to execute on the GPU is to wait for fence with **vkWaitForFences** this. Waiting for vkWaitForFences would allow you to stack multiple command buffer up to use the queue in parallel. This however requires more work and for 1 buffer transfer not even at run time **vkQueueWaitIdle** which waits until the queue is idling is good enough.

```c++
vkFreeCommandBuffers(device, transferCommandPool, 1, &commandBuffer);
```

Finally you've got to free the command buffer.
##### Creating the vertex buffer with staging buffers
Here is a code snippet on how create the vertex buffer using staging buffers copy buffer.
```c++
createBuffer(bufferSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT, 
VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, stagingBuffer, stagingBufferMemory);

void* data;
vkMapMemory(device, stagingBufferMemory, 0, bufferSize, 0, &data);
memcpy(data, vertices.data(), size_t(bufferSize));
vkUnmapMemory(device, stagingBufferMemory);

VkBuffer vertexBuffer;
VkDeviceMemory vertexBufferMemory;

createBuffer(bufferSize, VK_BUFFER_USAGE_TRANSFER_DST_BIT | VK_BUFFER_USAGE_VERTEX_BUFFER_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, vertexBuffer, vertexBufferMemory);

copyBuffer(stagingBuffer, vertexBuffer, bufferSize);

vkDestroyBuffer(device, stagingBuffer, nullptr);
vkFreeMemory(device, stagingBufferMemory, nullptr);
```

1) We create the staging buffer
2) We load memory into the staging buffer
3) We create the vertex buffer
4) We copy the staging buffer data to the vertex buffer which happens on the GPU
5) We destroy the staging buffers

The **createBuffer** second and third arguments contains the 
2) memory usage
3) memory type/location. 

These sets up what the buffer does and where the buffer lives.
##### What is an index buffer
These are pretty simply a buffer that contains pointer which are just a set of number which tells vulkan to reuse vertices. This means that we don't repeat over vertices, we just reuse ones we have already ran the vertex shader on.
##### Setting up the index buffer
The simplest use of a index buffer would be drawing a rectangle with 2 triangles. Of course in real life you're get this data from a model file rather than this. I added this here because set up will be easier to understand.

```c++
struct Vertex 
{
    vec2s position;
    vec3s colour;
};

const std::vector<Vertex> vertices = 
{
    {{ -0.5f, -0.5f }, { 1.f, 0.f, 0.f }},
    {{  0.5f, -0.5f }, { 0.f, 1.f, 0.f }},
    {{  0.5f,  0.5f }, { 0.f, 0.f, 1.f }},
    {{ -0.5f,  0.5f }, { 1.f, 1.f, 1.f }}
};

const std::vector<uint16_t> indices = 
{
    0, 1, 2, 2, 3, 0
};
```

Index buffer will either be uint16_t or uint32_t depending on how many indices there are in the index buffer.
##### Creating the index buffer with staging buffers

This will be basically the same as [[Creating the vertex buffer with staging buffers]] with like 2 differences

```c++
VkBuffer indexBuffer;
VkDeviceMemory indexBufferMemory;
VkBuffer stagingBuffer;
VkDeviceMemory stagingBufferMemory;

VkDeviceSize indexBufferSize = sizeof(indices[0]) * indices.size();

createBuffer(indexBufferSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT,
VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, stagingBuffer, stagingBufferMemory);

void* indexData;
vkMapMemory(device, stagingBufferMemory, 0, indexBufferSize, 0, &indexData);
memcpy(indexData, indices.data(), size_t(bufferSize));
vkUnmapMemory(device, stagingBufferMemory);

createBuffer(indexBufferSize, VK_BUFFER_USAGE_TRANSFER_DST_BIT | VK_BUFFER_USAGE_INDEX_BUFFER_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, indexBuffer, indexBufferMemory);

copyBuffer(stagingBuffer, indexBuffer, indexBufferSize);

vkDestroyBuffer(device, stagingBuffer, nullptr);
vkFreeMemory(device, stagingBufferMemory, nullptr);
```

The only differences here are the **indexBufferSize** which is equal to the **indices** which is were the uint16_t list of numbers live. When we create the index buffers with **createBuffer** we use an **VK_BUFFER_USAGE_INDEX_BUFFER_BIT** 

Like the vertex buffer you need to destroy them too.

```c++
vkDestroyBuffer(device, indexBuffer, nullptr);
vkFreeMemory(device, indexBufferMemory, nullptr);
```
##### Drawing with an index buffer
First thing to note when using a index buffer you can bind more than one at a time. You can use different index buffer for different vertex attributes, if you want to do that you'll need to duplicate the vertex data even if one attribute is different.

Knowing that you'll want to bind our index buffer

```c++
vkCmdBindVertexBuffers(commandBuffers[currentFrame], 0, 1, vertexBuffers, offsets);
vkCmdBindIndexBuffer(commandBuffers[currentFrame], indexBuffer, 0, VK_INDEX_TYPE_UINT16
```

The arguments go
1) The command buffer
2) The index buffer
3) The byte offset into the index buffer
4) The data type which can either VK_INDEX_TYPE_UINT16 or VK_INDEX_TYPE_UINT32

Next we need a new draw call to use the index buffer
```c++
vkCmdDrawIndexed(commandBuffers[currentFrame], uint32_t(indices.size()), 1, 0, 0, 0);
```

The arguments go
1) The command buffer
2) The number of indices
3) The number of instances (if you're not instance rendering set this to 1)
4) The offset into the index buffer (if you start at 1 for example the first index will be skipped)
5) The offset to add to the indices in the buffer
6) The offset for instancing

##### Making vertex and index buffer faster with aliasing
In the example of [[Creating the vertex buffer with staging buffers]] and [[Creating the index buffer with staging buffers]] we have used a vertex buffer, allocated memory to it, bound it and the same for the index buffer. The problem is we are repeating the allocation for each buffer. 

Vulkan allows you to offset into buffers for different usages. Why not do that for the vertex and index buffer, this reduces expensive CPU allocations and increases GPU cache locality. With this method there is more for the programmer to keep track off, you'll have to keep track of the offsets and make things at set up and draw time both match.

Set up similar as before

```c++
VkDeviceSize vertexbufferSize = sizeof(vertices[0]) * vertices.size();
VkDeviceSize indexBufferSize = sizeof(indices[0]) * indices.size();
VkDeviceSize totalBufferSize = vertexbufferSize + indexBufferSize;

VkBuffer stagingBuffer;
VkDeviceMemory stagingBufferMemory;

createBuffer(totalBufferSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT,
VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, stagingBuffer, stagingBufferMemory);

void* data;
vkMapMemory(device, stagingBufferMemory, 0, vertexbufferSize, 0, &data);
memcpy(data, vertices.data(), size_t(vertexbufferSize));
vkUnmapMemory(device, stagingBufferMemory);

void* indexData;
vkMapMemory(device, stagingBufferMemory, vertexbufferSize, indexBufferSize, 0, &indexData);
memcpy(indexData, indices.data(), size_t(indexBufferSize));
vkUnmapMemory(device, stagingBufferMemory);

VkBuffer modelBuffer;
VkDeviceMemory modelBufferMemory;

createBuffer(totalBufferSize, VK_BUFFER_USAGE_TRANSFER_DST_BIT | VK_BUFFER_USAGE_VERTEX_BUFFER_BIT | VK_BUFFER_USAGE_INDEX_BUFFER_BIT,
VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, modelBuffer, modelBufferMemory);

copyBuffer(stagingBuffer, modelBuffer, totalBufferSize);

vkDestroyBuffer(device, stagingBuffer, nullptr);
vkFreeMemory(device, stagingBufferMemory, nullptr);
```


The we allocate 1 staging buffer for everything which takes in the entire size off the buffer with the variable **totalBufferSize**.

The next difference is we actually use the offset variables when allocating memory into the buffers. you can see that **vkMapMemory** third argument **vertexbufferSize** we are putting the index buffer after the vertex data in the staging buffers memory. Ideally you would reduce this even further with by having the index data and the vertex attributes in one container.

Finally we when we create the **modelBuffer** we make it a VK_BUFFER_USAGE_VERTEX_BUFFER_BIT | VK_BUFFER_USAGE_INDEX_BUFFER_BIT for both types.

Lets use these then
```c++
VkBuffer vertexBuffers[] = { modelBuffer };
VkDeviceSize offsets[] = { 0 };
vkCmdBindVertexBuffers(commandBuffers[currentFrame], 0, 1, vertexBuffers, offsets);
vkCmdBindIndexBuffer(commandBuffers[currentFrame], modelBuffer, vertexbufferSize, VK_INDEX_TYPE_UINT16);
```

This is pretty much the same but we know that the vertex buffer data is first so there is an offset of 0. However when we bind the index buffer we offset the buffer by the size of the vertex buffer with **vertexbufferSize** to make sure we read all the indices.

This is the simplest case, so if you're loading in model data please be aware of the buffer offsetting while doing this.