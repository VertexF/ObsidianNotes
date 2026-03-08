2026-02-24 12:01
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]] [[buffer device address]]
# What is Buffer Device Address (BDA)

A buffer device address is an extension made feature in vulkan 1.2 that allows you to access the part of the GPU that holds the VkBuffers data with a pointer. This skips the need to use descriptor sets for buffers, meaning this is the other half of going bindless.

It allows you to update the device memory buffers with just a memcpy if it's CPU accessible. It's similar to an SSBO and if you want to remove descriptor set layouts completely with vulkan you can't use UBOs anywhere it has to be a pseudo-SSBO like structures.

You access the data in the shaders with a 64-bit integer (8 bytes) so basically a pointer. You're job after creating the buffer is getting the buffer device address to the shader were the buffer is used. This can be done with a push constants as you have 128 bytes of memory to work with and 256 bytes on vulkan 1.4 
# References
##### Main Notes
[[Creating a BDA buffer]]
#### Source Notes
