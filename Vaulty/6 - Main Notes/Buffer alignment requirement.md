2025-11-21 19:22
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]] [[vulkan shaders]]
# Buffer alignment requirement

Buffers in shaders need to be aligned with the C++ counter parts or the data gets corrupted on the shader. The basic idea is that the SPIRV add padding to the buffers to make things run faster on the GPU and this padding can misaligned things when the descriptor set tries to update the shader.

There are different rules to this depending on what alignment you're using. I've used **std140** in the past, which makes it so it follows a consistent rule.

With **std140** The GPU wants to organise memory into chunks big enough for 4 floats. If something can't fit in a these chunks it moves down to the next 4 chunks. 

The only things you need to remember are
1) Floats, ints, and bool take up 1 float chunk and can appear anywhere. 
2) vec2 appear in the beginning or the end of 2 chunk types.
3) vec3 takes up 3 chunks but can only appear at the beginning.
4) Anything takes up the maximum amount of space, starting from a from the beginning of a chunk. This may include adding a load of padding so the alignment works.
5) Matrices treated like array and pad the parts they don't use. So a mat2 takes up 4 chunks and pads the rest of the space to make it fill up a total of 8 chunks. 

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
# References
##### Main Notes
#### Source Notes
[[Uniform buffers]]
