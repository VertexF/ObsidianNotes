# Reference https://learnopengl.com/Advanced-OpenGL/Cubemaps

A cube map is just a bunch of textures rolled into one. Each of these six 2D make up the faces of a cube. Why a cube? It allows use to index and sample over each texture face with a direction vector.

Image you have a 1x1x1 cube that just textures. with an orgin we can sample these textures with a direction vector that looks like this:

![[cubemaps_sampling.png]]

The magnitude of doesn't really matter as long as the direction is supplied. Vulkan can retrieve the corresponding texels that the direction hits eventually and returns the properly sampled texture value.

If we have a cube to work with in the shader, we map the cube texture to that cube. The point at which the direction vector his the cube would be the same as the local interpolated value of the vextex attribute of the cube. The important part of this is to make sure that the direction vector stays at the origin of the cube. Meaning that the vertex positions of the cube itself is as texture coordinates. We just sample over these positions vertices with the direction vector which gets use the interpolated texel we want. This all means that we can easily get the correct face of the cubemap.

When you are creating the sampler for a cube texture you need to make sure that the all three components of the uvw coordinates are set up.

```c++
TextureResource* skyboxTextureResource = renderer.createSkybox(cubemapsImage, "SpaceCubeMap");
SamplerCreation skyboxSamplerCreation{};
skyboxSamplerCreation.minFilter = VK_FILTER_LINEAR;
skyboxSamplerCreation.magFilter = VK_FILTER_LINEAR;
skyboxSamplerCreation.addressModeU = VK_SAMPLER_ADDRESS_MODE_REPEAT;
skyboxSamplerCreation.addressModeV = VK_SAMPLER_ADDRESS_MODE_REPEAT;
skyboxSamplerCreation.addressModeW = VK_SAMPLER_ADDRESS_MODE_REPEAT;
SamplerHandle skyboxSampler = gpu.createSampler(skyboxSamplerCreation);
gpu.linkTextureSampler(skyboxTextureResource->handle, skyboxSampler);
```

For bindless textures you'll want to do something like this. Note the use of **samplerCube**

```c
#version 450

#extension GL_EXT_scalar_block_layout : require
// Bindless support
// Enable non uniform qualifier extension
#extension GL_EXT_nonuniform_qualifier : require

layout(scalar, set = 0, binding = 0) uniform MaterialConstant
{
    uint skyboxIndex;
};

layout(set = 1, binding = 0) uniform sampler2D globalTextures[];
//Alias textures to use the same binding point, as bindless texture is shared
//between all kind of textures: 1d, 2d, 3d.
layout(set = 1, binding = 0) uniform sampler3D globalTextures3D[];

layout(set = 1, binding = 0) uniform samplerCube globalTexturesCube[];

//Write only image do not need formatting in layout.
layout(set = 1, binding = 1) writeonly uniform image2D globalImages2D[];


layout (location = 0) in vec3 dir;

layout (location = 0) out vec4 fragColour;

void main()
{
	fragColour = texture(globalTexturesCube[nonuniformEXT(skyboxIndex)], dir);
}
```

A very important part of rendering a skybox is making sure that you disable depth testing and rendering it first. This means that everything gets rendered on top of the skybox.

