# Reference https://vulkan-tutorial.com/Uniform_buffers/Descriptor_set_layout_and_buffer

##### What are descriptors
These guys are used to update shaders every frame with some global variable. These can be buffers and images. An example usage would be providing the model view projection matrix in a buffer. 

To use these you need to set up these three things:
1) You need to specify a descriptor set layout during pipeline creation in the [[Setting up an empty pipeline layout]] stage of the pipeline.
2) You need to allocate descriptors like command buffer from a memory pool, called a descriptor pool.
3) Bind the descriptor you want to use during rendering.

##### What is a descriptor set layout
These are similar to render passes in the sense a render pass tells the type of attachments that are going to be used, and this is given to graphics pipeline when you're setting things up. Descriptor set layouts tells the graphics pipeline what resources are going to be accessed by the graphics pipeline.

These handle things like binding and what stage will the descriptors will be used in. 
##### What is a descriptor set
The descriptor set is similar to a framebuffer in some ways, as the framebuffer specifies the the VkImageView that will be bound to the render pass. The descriptor set specifies the actual buffer or image that will be bound to descriptors.

There are different types of descriptor sets such SSBO (Shader Storage Buffer Object) and UBO (Uniform Buffer Object).
##### Uniform buffer objects in vertex shaders
This will be a super basic example of how to use a uniform buffer object to update the vertex shader position to calculate perspective.

```c++
#version 450

layout (binding = 0) uniform ModelData
{
	mat4 model;
	mat4 view;
	mat4 proj;
}modelData;

layout (location = 0) in vec2 inPosition;
layout (location = 1) in vec3 inColour; 

layout(location = 0) out vec3 fragColour;

void main()
{
	gl_Position = modelData.proj * modelData.view * vec4(inPosition, 0.0, 1.0);
	fragColour = inColour;
}
```

The `binding = 0` is similar to the `location = 0` directive. We will reference the binding directly in the pipeline set up with the descriptor set layout. We need to do this for every single location.

So lets create it
```c++
VkDescriptorSetLayoutBinding uboLayoutBinding{};
uboLayoutBinding.binding = 0;
uboLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
uboLayoutBinding.descriptorCount = 1;
```

**.binding** and **.descriptorType** for telling vulkan what we are using the descriptor for and at what binding. We are making our binding a **VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER** so a uniform buffer object. The descriptor count tells vulkan how many values are in the array. For example, in skinning you can have a load of MVP matrices that are all different that need to be uploaded if you're doing animation on the CPU. For the simplest case of upload a 1 MVP we set this to 1.

```c++
uboLayoutBinding.stageFlags = VK_SHADER_STAGE_VERTEX_BIT;
```

This specified where it will be used. You can OR a bunch of these together or make it used in every shader step with **VK_SHADER_STAGE_ALL_GRAPHICS**.

```c++
uboLayoutBinding.pImmutableSamplers = nullptr;
```

These are used for image sampler related to descriptors.

All the descriptor set layouts are rolled into a **VkDescriptorSetLayout** and this goes into the graphics pipeline. So lets create the **VkDescriptorSetLayout**

```c++
VkDescriptorSetLayoutCreateInfo layoutInfo{};
layoutInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_LAYOUT_CREATE_INFO;
layoutInfo.bindingCount = 1;
layoutInfo.pBindings = &uboLayoutBinding;

VkDescriptorSetLayout descriptorSetLayout;
if (vkCreateDescriptorSetLayout(device, &layoutInfo, nullptr, &descriptorSetLayout) != VK_SUCCESS)
{
    printf("Failed to create descriptor set layout");
}
```

**.pBindings** and **.bindingCount** are used to handle arrays but if you have 1 you just do the typical.
##### Setting up the pipeline layout with a descriptor
To create a pipeline layout that uses descriptors you first need to have finished [[Creating a descriptor set layout]]  

```c++
VkPipelineLayout pipelineLayout;
VkPipelineLayoutCreateInfo pipelineLayoutCreateInfo{};
pipelineLayoutCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO;
pipelineLayoutCreateInfo.setLayoutCount = 1;
pipelineLayoutCreateInfo.pSetLayouts = &uboLayoutBinding;

if (vkCreatePipelineLayout(device, &pipelineLayoutCreateInfo, nullptr, &pipelineLayout) != VK_SUCCESS) 
{
	printf("Failed to create the layout pipeline.\n");
}
```

This is extremely similar to [[Setting up an empty pipeline layout]] but we are just adding in our descriptor set layout we created eariler.
##### Uniform buffer
Uniform buffer are a little different if you're planning to use them every frame it doesn't make sense to actually stage them because we are calculating stuff per frame on the host, you'll want make it host visible and **NOT** use staging buffers. You instead want to leave the buffer open to have data written to it every frame with **vkMapMemory**

As we have muiltple frames in flight that will be using the buffer we also need to have a buffer data for each frame. 

```c++
std::vector<VkBuffer> uniformBuffers;
std::vector<VkDeviceMemory> uniformBuffersMemory;
std::vector<void*> uniformBuffersMapped;
```

Then we create the buffers with the **createUniformBuffers** function which takes our vector grows them to size and create a host visible buffer with our limited [[Making a basic createBuffer function|createBuffer]] function

```c++
static void createUniformBuffers()
{
    VkDeviceSize bufferSize = sizeof(ModelData);

    uniformBuffers.resize(imageCount);
    uniformBuffersMemory.resize(imageCount);
    uniformBuffersMapped.resize(imageCount);

    for (uint32_t i = 0; i < imageCount; ++i)
    {
        createBuffer(bufferSize, VK_BUFFER_USAGE_UNIFORM_BUFFER_BIT, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, uniformBuffers[i], uniformBuffersMemory[i]);

        vkMapMemory(device, uniformBuffersMemory[i], 0, bufferSize, 0, &uniformBuffersMapped[i]);
    }
}
```

Here each buffer is mapped with **vkMapMemory** for the entire life time of the program. By not unmapping the buffer we are doing something called persistent mapping, this means we have a pointer in **uniformBuffersMemory** vector we can write later on. This increase performance over re-mapping over and over every frame as mapping memory isn't free.

Like all other buffers we need to destroy them
```c++
for (uint32_t i = 0; i < maxFramesInFlight; ++i) 
{
    vkDestroyBuffer(device, uniformBuffers[i], nullptr);
    vkFreeMemory(device, uniformBuffersMemory[i], nullptr);
}

vkDestroyDescriptorSetLayout(device, descriptorSetLayout, nullptr);
```
##### Updating uniform data
In this example we are updating the perspective of the vertices in the vertex shader. I wont explain any of the non-vulkan stuff apart from we have a camera here, perspective and model matrix we ar going to write

```c++
auto startTime = std::chrono::high_resolution_clock::now();

		///... reset the fence and acquire the next image.
		
auto currentTime = std::chrono::high_resolution_clock::now();
float time = std::chrono::duration<float, std::chrono::seconds::period>(currentTime - startTime).count();
ModelData modelData{};
modelData.model = glms_rotate(glms_mat4_identity(), time * glm_rad(90.f), { 0.f, 0.f, 1.f });
modelData.view = glms_lookat({ 2.f, 2.f, 2.f }, { 0.f, 0.f, 0.f }, { 0.f, 0.f, 1.f });
modelData.proj = glms_perspective(glm_rad(45.f), swapchainExtent.width / (float)swapchainExtent.height, 0.1f, 10.f);
modelData.proj.m11 *= -1;

memcpy(uniformBuffersMapped[currentFrame], &modelData, sizeof(modelData));
```

With our **memcpy** we are writing to our mapped memory with that pointer we left exposed.

Pass data tiny amounts of data to the using UBO isn't the most efficent. You'll want to use push constants to get this working. You'll need to be aware of the pros and cons of all buffer types to know which one to use.
##### Using descriptor sets
To use descriptor set you need to specify the [[Creating a descriptor set layout|descriptor set layout]] and [[Creating uniform buffers|create buffers]] or images to use within the descriptor sets. You then need to make a descriptor pool, create the descriptor sets from that pool, update them by writing the data you want the descriptor sets to handle and finally bind that bad boy to run along side your draw call. 
##### Creating a descriptor pool
You can't just make descriptor sets, they are similar to command buffer, they need a descriptor pool to work. 

```c++
VkDescriptorPoolSize poolSize{};
poolSize.type = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
poolSize.descriptorCount = imageCount;

VkDescriptorPoolCreateInfo poolDescriptorCreateInfo{};
poolDescriptorCreateInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_POOL_CREATE_INFO;
poolDescriptorCreateInfo.poolSizeCount = 1;
poolDescriptorCreateInfo.pPoolSizes = &poolSize;
```

First thing we need to is specify with **VkDescriptorPoolSize** how many descriptor set we want with **.descriptorCount** we are going create one descriptor set per frame so we will use the **imageCount**. Next we want to tell vulkan what the descriptor set will be handling using **.type** I want uniform buffers in my descriptor sets so I set this to **VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER** 

Next we pass that over to the **VkDescriptorPoolCreateInfo** and begin setting that up.

```c++
poolDescriptorCreateInfo.maxSets = imageCount;
```

We also need to specify the amount the **MAX** amount of descriptor sets. Again we know how many we want to allocate so this it set the **imageCount**.

You can set this optional **.flag** with something
```c++
poolDescriptorCreateInfo.flags = VK_DESCRIPTOR_POOL_CREATE_FREE_DESCRIPTOR_SET_BIT;
```

This tells the descriptor set if individual descriptors can be freed or not. If you're just using the simplest set up which is just simply creating them and **NOT** touching them apart from the data underneth then you **DON'T** need this. I left to 0.

```c++
VkDescriptorPool descriptorPool;
if (vkCreateDescriptorPool(device, &poolDescriptorCreateInfo, nullptr, &descriptorPool) != VK_SUCCESS)
{
    printf("Failed to create descriptor pool");
}
```

Then you can simply create your descriptor pool.
##### Creating Descriptor set
If you've set up the descriptor pool you can now set up the descriptor sets. You need to allocate memory to them like this

```c++
//Descriptor sets creation
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

Then we can allocate a set of descriptor sets. The function **vkAllocateDescriptorSets** expect there to be a identical set of descriptor set layouts in the **descriptorAllocateInfo** struct which is why we had to copy them.

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
    VkWriteDescriptorSet descriptorWrite{};
    descriptorWrite.sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
    descriptorWrite.dstSet = descriptorSets[i];
    descriptorWrite.dstBinding = 0;
    descriptorWrite.dstArrayElement = 0;
    descriptorWrite.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
    descriptorWrite.descriptorCount = 1;
    descriptorWrite.pBufferInfo = &desBufferInfo;
}
```

The parameters **.dstSet** and **.dstBinding** specify the descriptor set you want to update and the binding. Descriptor sets can be arrays we so **.dstArrayElement** tells vulkan what is the first index of that array, if you're not using arrays set the to 0. 

**.descriptorType** specify the descriptor type again. It's possible to update arrays of descriptor sets at the same time from **.dstArrayElement** to **.descriptorCount** If you don't have an array just set **.descriptorCount** to 1.

```c++
descriptorWrite.pBufferInfo = &desBufferInfo;
descriptorWrite.pImageInfo = nullptr;
descriptorWrite.pTexelBufferView = nullptr;
```

Then you add what buffer type/ image you want. 
- **.pBufferInfo** is for buffers
- **.pImageInfo** is for image data
- **.pTexelBufferView** is for a buffer view.

We are using a regular buffer so **.pbufferInfo** it is.

```c++
vkUpdateDescriptorSets(device, 1, &descriptorWrite, 0, nullptr);
```

Then run the update function to update your descriptors, remember this can handle arrays so we pass in the reference with a size of 1. We are passing in the **VkWriteDescriptorSet** which writes data, you can also pass in **VkCopyDescriptorSet** this copies descriptor sets to another set.
##### Using the descriptor sets

After creating the descriptor sets you'll need to run this before any **vkCmdDraw...** 

```c++
vkCmdBindDescriptorSets(commandBuffers[currentFrame], VK_PIPELINE_BIND_POINT_GRAPHICS, pipelineLayout, 0, 1, &descriptorSets[currentFrame], 0, nullptr);
```

2) Descriptor sets can be used in compute pipelines as well as graphics pipelines so set that with **VK_PIPELINE_BIND_POINT_GRAPHICS** 
3) Next you need to specify the pipeline layout that the descriptor set is based on.
4) The next three are to do with the index of the descriptor. So the forth argument is the telling vulkan what the first index is.
5) This tells vulkan how many to bind at a time.
6) This tells vulkan the actually array of descriptor sets to bind. 
7) The next two are to do with dynamic descriptors and the offset into them. We aren't using them so the dynamic count is 0
8) The dynamic descriptor array of uint32_t which are the offsets is a nullptr if you're not using this feature.
##### Buffer alignment requirements
Buffers in shaders need to be aligned with the C++ counter parts or the data gets corrupted on the shader. The basic idea is that the SPIRV add padding to the buffers to make things run faster on the GPU and this padding can mis-align things when the descriptor set tries to update the shader.

There are different rules to this depending on what alignment you're using. I've used **std140** in the past, which makes it so it follows a consistent rule.

The GPU wants to organise memory into chunks big enough for 4 floats. If something can't fit in a these chunks it moves down to the next 4 chunks. 

The only things you need to remember are
1) like floats, ints, and bool take up 1 float chunk and can appear anywhere. 
2) vec2 appear in the beginning or the end of 2 chunk types.
3) vec3 takes up 3 chunks but can only appear at the beginning.
4) Anything takes up the maximum amount of space, starting from a from the beginning of a chunk. This may include adding a load of padding so the alignment works.
5) Matrices treated like array and pad the parts they don't use. So a mat2 takes up 2 chunks and padd the rest of the space to make it fill up a total 8 chunks. 

More information can be found on this excellent tutorial on WebGL https://youtu.be/JPvbRko9lBg?si=F8m8Mj1fme3ewlBR&t=196 to 8:58

This is why we align by 16 in our code because 4 x 4 = 16 meaning that's how many bytes we are working with.
```c++
struct ModelData
{
    alignas(16) mat4s model;
    alignas(16) mat4s view;
    alignas(16) mat4s proj;
};
```

This already is aligned but if I add other types that wont be true.
##### Multiple descriptor sets
You can bind more than 1 descriptor set. You still need to specify a descriptor set layout for each descriptor. The shader references a specific shader with the **set** declarative. 

```c++
layout(set = 0, binding = 0) uniform modelData {  ...  };
```

The idea is that you use this to put descriptor that vary per-object and descriptors that are shared into there own descriptor sets. So a basically it's just an array of descriptor sets that can be accessed individually in shaders. This can be performant as there is less binding that needs to happen per draw call.

