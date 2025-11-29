2025-11-28 09:30
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]]
# Creating a sampler

We are going to use this function to create a texture.

```c++
static VkSampler createSampler() { /*stuff here*/ }
```

We begin by making the sampler creation struct.
```c++
VkSamplerCreateInfo samplerInfo{};
samplerInfo.sType = VK_STRUCTURE_TYPE_SAMPLER_CREATE_INFO;
samplerInfo.magFilter = VK_FILTER_LINEAR;
samplerInfo.minFilter = VK_FILTER_LINEAR;
```

The **.magFilter** and **.minFilter** states how the texels will be interpolated with either magnification with is concerned with oversampling and minified for undersampling. 
- **VK_FILTER_LINEAR** is for linear interpolation
- **VK_FILTER_NEAREST** is for no interpolation.

```c++
samplerInfo.addressModeU = VK_SAMPLER_ADDRESS_MODE_REPEAT;
samplerInfo.addressModeV = VK_SAMPLER_ADDRESS_MODE_REPEAT;
samplerInfo.addressModeW = VK_SAMPLER_ADDRESS_MODE_REPEAT;
```

These are to do with how the texels are handled if you go outside of the texture image. As we are texture space the coordinate axes are U, V, W instead of X, Y, Z. Meaning you can have a different sampler modes for different axes.

You have these 5 to choose from.
- **VK_SAMPLER_ADDRESS_MODE_REPEAT**
- **VK_SAMPLER_ADDRESS_MODE_MIRRORED_REPEAT**
- **VK_SAMPLER_ADDRESS_MODE_CLAMP_TO_EDGE** 
- **VK_SAMPLER_ADDRESS_MODE_CLAMP_TO_BORDER**
- **VK_SAMPLER_ADDRESS_MODE_MIRROR_CLAMP_TO_EDGE**

For more information on what this looks like look at [[What are samplers]] note.

```c++
samplerInfo.anisotropyEnable = VK_TRUE;
samplerInfo.maxAnisotropy = maxAnisotropy;
```

When **.anisotropyEnable** is true this allows use to increase the amount of texel samples that can be used for calculating the final image. This will increase the quality of the image but reduce performance. The overhead seems to be rather low so it's a recommended features to use.

However we also need to know what the **.maxAnisotropy** and to turn on that feature on the graphics card before we can use this. **maxAniostropy** is a uint32_t

First thing we need to do when selecting an appropriate GPU is get the max aniostropy from our selected GPU
```c++
VkPhysicalDeviceProperties properties;
vkGetPhysicalDeviceProperties(gpus[i], &properties);

//... score being checked here for the best GPU

if (highestScore < score) 
{
    highestScore = score;
    bestGPUIndex = i;
    maxAnisotropy = properties.limits.maxSamplerAnisotropy;
}
```

After that we will need to enable this anisotropy feature on the GPU with the **VkPhysicalDeviceFeatures2** struct.

```c++
deviceFeatures.features.samplerAnisotropy = VK_TRUE;
```

Not all graphics will support this so you'll need to check in the but for now, I will again assume that most people running this application will have a graphics card made in the last 10 years

```c++
samplerInfo.anisotropyEnable = VK_FALSE;
samplerInfo.maxAnisotropy = 1.f;
```

The code above will turn off anisotropy.

```c++
samplerInfo.borderColor = VK_BORDER_COLOR_INT_OPAQUE_BLACK;
```

This tells the sampler to add a border when sampling beyond the clamped edge when **VK_SAMPLER_ADDRESS_MODE_CLAMP_TO_BORDER** address mode is used. You can do black, white, transparent or a custom colour with a vulkan extension.

```c++
samplerInfo.unnormalizedCoordinates = VK_FALSE;
```

If this is set to true, you can sample from `[0, textureWidth)` and `[0, textureHeight)` you almost never want this because you'll often be working with textures of different sizes so normalised to `[0, 1)` and `[0, 1)` is best.

```c++
samplerInfo.mipmapMode = VK_SAMPLER_MIPMAP_MODE_LINEAR;
samplerInfo.mipLodBias = 0.f;
samplerInfo.minLod = 0.f;
samplerInfo.maxLod = 0.f;
```

These are all to do with mipmapping if you're not mipmapping this is the set up you'll want to have.

```c++
VkSampler sampler;
if (vkCreateSampler(device, &samplerInfo, nullptr, &sampler) != VK_SUCCESS) 
{
    printf("Could not create sampler");
}
```

The sampler can be applied to any VkImage you want. That's why the VkImage and VkSampler are different objects.

Don't forget to destroy the sampler.

```c++
vkDestroySampler(device, textureSampler, nullptr);
```
# References
##### Main Notes
[[What are samplers]]
#### Source Notes
[[Texture mapping]]