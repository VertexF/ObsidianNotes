2025-12-02 19:52
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]] [[vulkan shaders]]
# Using textures within a shader

The first thing you need to do is be aware the vertex attribute that comes through the vertex buffer needs to handle. [[Creating basic texture coordinate]] So you need to pass the texture coordinates from the vertex shader to the fragment shader.

```c
#version 450
layout (location = 0) in vec2 inPosition;
layout (location = 1) in vec3 inColour;
layout (location = 2) in vec2 inTexCoord; 

layout (location = 0) out vec3 fragColour;
layout (location = 1) out vec2 fragTexCoord;

void main()
{
	gl_Position = modelData.proj * modelData.view * modelData.model * vec4(inPosition, 0.0, 1.0);
	fragColour = inColour;
	fragTexCoord = inTexCoord;
}
```

Here we are just going to print the texture coordinate with the texture coordinates we passed over from the vertex shader.
```c
layout(location = 0) in vec3 fragColour;
layout(location = 1) in vec2 fragTexCoord;

layout(location = 0) out vec4 outColour;

void main()
{
	outColour = vec4(fragTexCoord, 0.0, 1.0);
}
```

If you're having a problem with textures on models you can actually just display the texture coordinates directly. This can be thought of as shader "printf" debugging as it shows you if the texture coordinates make sense.

![[texture_coordinates.png]]
This is what a correct texture coordinates should look like. The green represents the hoziontal coordinates and the red channel represents the vertical coordinates. The yellow and the black corners show that we are interperlating correctly from 0, 0 to 1, 1 across the square.

You can see that if I make a mistake with the texture coordinates I don't get this picture.

![[texture_coordinate_wrong.png]] 

Since we are missing the black corner you can get an idea on what's wrong here.

To actually use the combined image sampler you'll want to create a uniform **sampler** object in the shader you want to use a texture.

```c
layout(binding = 1) uniform sampler2D texSampler;
```

You have 1D and 3D version of this so be aware on what type of texture you're interested in. The binding has to match the `descriptorWrites[1].dstBinding = 1;` when we created the `std::array<VkWriteDescriptorSet, 2>` when binding the image buffer.

```c
vec4 tex = texture(texSampler, fragTexCoord);
outColour = vec4(fragColour * tex.rgb, tex.a);
```

Next you'll want to use the **texture** function in your shader that deals with the sampling transformation and filtering in the background. 

I only added a little extra code here so I could add the **fragColour** to the RBG parts of the texture while leaving the alpha pixels alone.

```c
vec4 tex = texture(texSampler, fragTexCoord * 4.0);
outColour = vec4(fragColour * tex.rgb, tex.a);
```

You can if you want mess around the with texture coordinate values and you'll see the address mode at work for a given sampler. 

This is the output of that result
![[texture_frogx4.png]]
# References
##### Main Notes
[[Creating basic texture coordinate]]
#### Source Notes
[[Texture mapping]]
