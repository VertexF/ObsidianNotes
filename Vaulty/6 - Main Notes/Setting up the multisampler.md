2025-10-26 09:47
Status: #baby 
Tags: [[vulkan]] [[vulkan graphic pipeline]]
# Setting up the multisampler

Multisampling is a way to reduce aliasing effects. It takes the work of the fragement shader that have rastered to the same pixel and samples over that pixel multiple times. With more than 1 sample you can do some kind of sampling to smooth out edges.

To use this feature you'll need to turn on a GPU feature, if you don't want to use it you still need to build up the struct **VkPipelineMultisampleStateCreateInfo** and pass it to the graphics pipeline.

This is what a non-multisampler setup looks like
```c++
VkPipelineMultisampleStateCreateInfo multisamplingState{};
multisamplingState.sType = VK_STRUCTURE_TYPE_PIPELINE_MULTISAMPLE_STATE_CREATE_INFO;
multisamplingState.sampleShadingEnable = VK_FALSE;
multisamplingState.rasterizationSamples = VK_SAMPLE_COUNT_1_BIT;
multisamplingState.minSampleShading = 1.f; //optional
multisamplingState.pSampleMask = nullptr; //optional
multisamplingState.alphaToCoverageEnable = VK_FALSE; //optional
multisamplingState.alphaToOneEnable = VK_FALSE; //optional
```
# References
##### Main Notes
[[What is the graphics pipeline]]
#### Source Notes
[[Drawing a triangle]]