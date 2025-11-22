2025-11-21 11:06
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Updating uniform buffers

In this example we are updating the perspective of the vertices in the vertex shader. I wont explain any of the non-vulkan stuff apart from we have a camera here, perspective and model matrix we ar going to write

```c++
//NOTE: startTime is OUTSIDE the render loop.
auto startTime = std::chrono::high_resolution_clock::now();

		///... begin the rendering.
		
auto currentTime = std::chrono::high_resolution_clock::now();
float time = std::chrono::duration<float, std::chrono::seconds::period>(currentTime - startTime).count();
ModelData modelData{};
modelData.model = glms_rotate(glms_mat4_identity(), time * glm_rad(90.f), { 0.f, 0.f, 1.f });
modelData.view = glms_lookat({ 2.f, 2.f, 2.f }, { 0.f, 0.f, 0.f }, { 0.f, 0.f, 1.f });
modelData.proj = glms_perspective(glm_rad(45.f), swapchainExtent.width / (float)swapchainExtent.height, 0.1f, 100.f);
modelData.proj.m11 *= -1;

memcpy(uniformBuffersMapped[currentFrame], &modelData, sizeof(modelData));
```

With our **memcpy** we are writing to our mapped memory with that pointer we left exposed.

Passing tiny amounts of data to the using UBO isn't the most efficent. You'll want to use push constants to get this working as it works well with small amounts of memory. You'll need to be aware of the pros and cons of all buffer types to know which one to use.
# References
##### Main Notes
[[Creating uniform buffers]]
#### Source Notes
[[Uniform buffers]]
