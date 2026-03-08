2026-03-08 09:55
Status: #baby 
Tags: [[vulkan]] [[GPU hardware]] [[vulkan descriptors]]
# The problem with descriptor

There is a problem with descriptor set layouts they are too close to the memory layout of the shader. If you have constrain your engine works around like having 32 UBOs and 128 textures, you set up different bindings in your shader for each and everything is chill. You typically want one giant descriptor set to rule them all.

As soon as you want to use a different interface in your shader, everything breaks. If you want to avoid having rebinding descriptor sets over and over again you need different pipelines making handling pipelines in vulkan even harder. Now you have to now the pipeline layout before creating **ANY** pipelines. You can bind multiple materials together with [[Multiple descriptor sets]] but that's limited to vertex and fragment shaders, mixing materials doesn't work, which is what you really want.
# References
##### Main Notes
[[The GPU hardware of descriptors]]
[[What are descriptors]]
[[What are descriptor sets]]
[[What is a descriptor set layout]]
[[Multiple descriptor sets]]
[[Binding a descriptor set to use at draw time]]
#### Source Notes
[[Descriptors are hard]]
