2026-02-27 08:38
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]]
# What is bindless

When you are using bindless with textures you are still binding but you bind an list of resources that can be indexed into in the shaders. Basically every single desktop GPU support this, so use it. It makes using Vulkan easier to use. These resources will be mainly used for textures which all can be used across shaders. There is a limit to the size of textures that can in that array but it's large enough that it's still super useful.

You can go bindless for buffers but you honestly have to use something else like [VK_EXT_descriptor_heap](https://docs.vulkan.org/guide/latest/descriptor_heap.html) or [VK_EXT_descriptor_buffer](https://www.khronos.org/blog/vk-ext-descriptor-buffer)  I talk about the descriptor buffer version in the note [[What is Buffer Device Address (BDA)]] 
# References
##### Main Notes
[[Enabling bindless textures]]
[[Creating the bindless descriptor pool]]
[[What is Buffer Device Address (BDA)]]
#### Source Notes
[[Bindless - Mastering Graphics Programming with Vulkan]]