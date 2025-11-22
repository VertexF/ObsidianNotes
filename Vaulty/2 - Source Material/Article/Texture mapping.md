# Reference https://vulkan-tutorial.com/Texture_mapping/Images

Texture mapping is pretty complicated and requires good use of memory layout and transitions. The overall idea of adding a texture to some polygons is to:
- Create an image back by device memory
- Fill it with pixels from a file.
- Create an image sampler
- We'll use our descriptors we made to add to sample our colours from in the fragment shader.

It's similar to seting up a vertex buffer honestly, you just need to make a staging buffer, then transfer over the staged pixles into our final image, ready to be used.

