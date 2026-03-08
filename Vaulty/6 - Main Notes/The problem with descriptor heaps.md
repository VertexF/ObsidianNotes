2026-03-08 10:09
Status: #baby 
Tags: [[vulkan]] [[GPU hardware]] [[vulkan descriptors]]
# The problem with descriptor heaps

One things that Microsoft did was with it's D3D12 was to embrace descriptor heaps for their hardware and drivers. In the early days these had to be bound before you can execute any 3D work or compute commands. Shaders have the usual HLSL register notion for describing the descriptor interface. When the shader are compiled you get the descriptor tables used to map the bindings in the shader to ranges in the relevant descriptor heap. While the heap range itself is a fixed the application has access to move the range around at will. 

With SM6.6 in D3D12 Microsoft significantly improve flexibility and further embraced heaps. Instead of using this descriptor table in the root descriptor, applications can use heap indices directly. This provide access to the full bindless experience! The developers just have to worry about managing the heap allocation with the resources lifetimes and figure out how to get the indices into their shaders. Gone of the days of having to fiddle with the interface layouts because you wanted to change 1 things in a shader, now you're entire descriptor layout has to change. Developers loved this change honestly.

A descriptor heap is just a very restrictive buffer. Meaning for AMD drivers just use of their descriptor buffer bindings (resource and sampler heaps are separate in D3D12) and implements a heap as a descriptor buffer.

There is a downside to descriptor heap approach is that is forces some amount of extra indirection, especially with the SM6.6 bindless model. First thing you need to do is load a heap index into a constant buffer and pass that to loadOp/storeOp. This allows the GPU to do the sequence instruction that fetches the heap, after that it does some offset calculations and does the actual load and store operations for that corresponding pointer. This can actually slow things down if there are a lot of operations happening, the compiler can remove some but it's not a given that will happen if you have a tonne of stuff.
# References
##### Main Notes
[[Vulkan 1.0 descriptor set model]]
#### Source Notes
[[Descriptors are hard]]
