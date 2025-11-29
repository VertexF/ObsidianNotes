2025-11-27 11:51
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]]
# Introduction to texture mapping

Texture mapping is pretty complicated and requires good use of memory layout and transitions. The overall idea of adding a texture to some polygons is to:
- Create an image back with device memory
- Fill it with pixels from a file.
- Create an image sampler
- Then use something like descriptors to add a sampler to our colours in the fragment shader.

Yes we have already talking about this in [[What are image views]] and [[Creating an image view]] but we need to do more than we did to handle textures. It's similar to setting up a vertex buffer honestly, you just need to make a staging buffer, then transfer over the staged pixels into our final image, ready to be used.

We also need to worry about memory layouts on top of this. We seen some of these before so here a quick list.
1) **VK_IMAGE_LAYOUT_PRESENT_SRC** optimal for prensentation.
2) **VK_IMAGE_LAYOUT_COLOUR_ATTACHMENT_OPTIMAL** optimal as an attachment for writing from fragment shader.
3) **VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL** optimal for being the source in transfer operations.
4) **VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL** optimal for being the destination in transfer operations.
5) **VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL** optimal for sampling from a shader.
6) **VK_IMAGE_LAYOUT_GENERAL** can be used for anything but it's less performant for everything. You might need to use it in special cases like when an image is both input and out or reading an image after it have been left the preinitialised layout setup.

When handling texture and images that come from that you'll need to also use pipeline barriers. We used this for example in [[Setting the synchronisation with the subpass dependencies]] 
# References
##### Main Notes
[[Creating descriptor sets]]
[[Binding a descriptor set to use at draw time]]
[[What are image views]]
[[Creating an image view]]
[[Setting the synchronisation with the subpass dependencies]] 
#### Source Notes
[[Texture mapping]]