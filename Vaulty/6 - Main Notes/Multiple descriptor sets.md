2025-11-21 19:26
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]]
# Multiple descriptor sets

#### WARNING: My understanding on this topic is a little weak.

You can bind more than 1 descriptor set. You still need to specify a descriptor set layout for each descriptor. The shader references a specific shader with the **set** directive. 

```c++
layout(set = 0, binding = 0) uniform modelData {  ...  };
```

The idea is that you use this to put descriptors that vary per-object and descriptors that are shared into there own descriptor sets. So a basically it's just an array of descriptor sets that can be accessed individually in shaders. This can be performant as there is less binding that needs to happen per draw call.
# References
##### Main Notes
[[Binding a descriptor set to use at draw time]]
[[Creating descriptor sets]]
#### Source Notes
[[Uniform buffers]]