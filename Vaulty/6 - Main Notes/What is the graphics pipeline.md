2025-10-20 07:44
Status: #baby 
Tags: [[vulkan]] [[vulkan graphic pipeline]]
# What is the graphics pipeline

The graphics pipeline in vulkan is a the sequence of operation that takes vertices and textures of models and turns then into pixel.

The vulkan graphic pipeline set up the graphics pipeline used in any API. 

![[vulkan_simplified_pipeline.svg]]

It's important to note in vulkan the graphics pipeline is set up in one go in the set up part of the application and is completely immutable. Meaning the driver can do optimisation but with the restriction if you want a different shaders to be used or something like that, you'll need a seperate graphics pipeline. 
# References
##### Main Notes
[[Setting up the dynamic states]]
[[Setting up the vertex inputs]]
[[Setting up the input assembly]]
#### Source Notes
[[Drawing a triangle]]
