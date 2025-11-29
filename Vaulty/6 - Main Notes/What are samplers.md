2025-11-28 08:23
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]]
# What are samplers

You can send up an image to the shaders read directly from it, however that's never really done with textures. Typically you'll want sampler that applies filters and transforms that computes the final image.

Oversampling is when have some geometry that has more fragments than texels for a given texture. If you don't apply filtering you get the first one. With a sampler though you can linearly interpolate between the 4 closest texels you get the second picture.

![[texture_filtering.png]]

Undersampling is the opposite problem were you have too many texels for fragments and you get weird artifacts when you have sharp contrasting textures. You can apply anisotropic filtering to help with this.

![[anisotropic_filtering.png]]

Samplers also details with what happens when you sample outside the texture on geometry, in the samplers **address mode**. Here are pictures of the possible abilities

![[texture_addressing.png]]
# References
##### Main Notes
[[Creating a sampler]]
#### Source Notes
[[Texture mapping]]