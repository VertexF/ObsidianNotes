2026-06-12 09:06
Status: #baby 
Tags: [[graphics theory]] [[ray tracing]]
# Indirect light transport for rays

The recursive nature of ray tracing allows for indirect specular reflection. We do this by reflecting a ray at a surface normal at the intersection point if it's shiny. This can happen recursively, which all gets added back to the camera by following back the cameras ray eventually.

With hardware based ray tracing you can specify a limit to the amount of recusion you have in your pipeline set up. In vulkan it's under the `VkRayTracingPipelineCreationInfo::maxPipelineRayRecursionDepth`

If you do this in a compute shader, you'll need to maintain your own stack for the recursion.
# References
##### Main Notes
[[What is stochastic progressive phonotic mapping]]
#### Source Notes
[[Physically Based Rendering - Introduction]]
