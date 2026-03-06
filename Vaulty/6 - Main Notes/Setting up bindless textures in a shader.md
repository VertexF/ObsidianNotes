2026-02-27 09:34
Status: #baby 
Tags: [[vulkan]] [[vulkan shaders]] [[vulkan textures]]
# Setting up bindless textures in a shader

Now assuming you've done all the work to update a bindless descriptor set before the draw call this is what your shader should be trying to do.

This is a fragment shader using bindless textures.
```c
#extension GL_EXT_nonuniform_qualifier : require

layout(scalar, set = 0, binding = 1) uniform MaterialConstant
{
    uvec4 textures;
};

layout(set = 1, binding = 0) uniform sampler2D globalTextures[];
//Alias textures to use the same binding point, as bindless texture is shared
//between all kind of textures: 1d, 2d, 3d.
layout(set = 1, binding = 0) uniform sampler3D globalTextures3D[];

//Write only image do not need formatting in layout.
layout(set = 1, binding = 1) writeonly uniform image2D globalImages2D[];

#define PI 3.1415926538
#define INVALID_TEXTURE_INDEX 65535

layout(location = 0) in vec2 vTexcoord0;

layout(location = 0) out vec4 fragColour;

void main()
{
    mat3 TBN = mat3(1.0);
    fragColour = texture(globalTextures[nonuniformEXT(textures.x)], vTexcoord0);
}
```

Two things are important here. One that we are on a different set binding index to non-bindless descriptor set layouts. You can't have bindless and non-bindless mixing here.

Second is that when we need to access something out of array we need to use the `nonuniformEXT()` which means that different threads can access this array at any point. The extension is required too for update after bind which is part of how you update a bindless descriptor.

The rest is pretty straight forward the you index into the array using an integet **nonuniformEXT(textures.x)** and then you just sample into that texture with the **texture()** function.
# References
##### Main Notes
[[Binding a bindless descriptor set]]
[[Updating a bindless descriptor set]]
[[Creating a bindless descriptor set layout]]
[[Creating descriptor sets]] 
[[Creating the bindless descriptor pool]]
[[Creating an combined image sampler descriptor]]
#### Source Notes
[[Bindless - Mastering Graphics Programming with Vulkan]]