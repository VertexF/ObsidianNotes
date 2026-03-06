# Reference https://www.gfxstrand.net/faith/blog/2022/08/descriptors-are-hard/

There is a problem with descriptor set layouts they are too close to the memory layout of the shader. If you have contraints your engine works around like having 32 UBOs and 128 textures, you set up different bindings in your shader for each and everything is chill. You typically want one giant descriptor set to rule them all.

As soon as you want to use a different interface in your shader, everything breaks. If you want to avoid having rebinding descriptor sets over and over again you need different pipelines making handling pipelines in vulkan even harder. Now you have to now the pipeline layout before creating **ANY** pipelines. You can bind multiple materials together with [[Multiple descriptor sets]] but that's limited to vertex and fragment shaders, mixing materials doesn't work, which is what you really want.
##### The problem space
To understand problem you have to realise that Vulkan is targetting a tonne of hardware. So lets dig into the hardware.

There are 4 possible descriptor set implementations:

1) **Direct Access (D)** This is were the shader itself passes the entire thing over to the GPU to be access directly. 
2) **Descriptor Buffers (B)** The hardware has a piece of memory were a buffer lives. To access the descriptor the GPU needs to read that piece of memory to access the data. Meaning that instead of passing the entire descriptor, it lives in the buffer now. To access the buffer the buffer is lives either on a fixed address or the base address is passed to the shader.
3) **Descriptor Heap (H)** Descriptors of a particular type all live in a single global table or heap. Updating this is pretty expensive. However once we have the table created we then just send an index to loop up things in this table/heap.
4) **Fixed HW bindings (F)** This is when you have registers or sections of the hardware that fill up a slot. With the push towards bindless, these are only used for fixed function pipeline things. These are still around because 1.0 supported pre-bindless hardware.

| Hardware            | Textures | Images | Samplers | Boarder Colours | Typed Buffers | UBOs  | SSBOs |
| ------------------- | -------- | ------ | -------- | --------------- | ------------- | ----- | ----- |
| NVIDIA (Kepler+)    | H        | H      | H        |                 | H             | D/F   | H     |
| AMD                 | D        | D      | D        | H               | D             | D     | D     |
| Intel (Skylake+)    | H        | H      | H        |                 | H             | H/F/D | H/D   |
| Intel (pre-Skylake) | F        | F      | F        |                 | F             | D/F   | F     |
| Arm (Valhal+)       | B        | B      | B        |                 | B             | B/D/F | B/D   |
| Arm (Pre-Valhal)    | F        | F      | F        |                 | F             | F/D   | F     |
| Qualcomm (a5xx+)    | B        | B      | B        |                 | B             | B     | B     |
| Broadcom (vc5)      | D        | D      | D        |                 | D             | D     | D     |
**Intel (pre-Skylake)** is little more flexible than this table makes out. It uses a heap like structure but has a level of indirection that is limited to 240. **Post-skylake** it uses a different set up but still has the heap thing but it allows a backdoor to do **Descriptor Heap** stuff.
##### Vulkan 1.0 Descriptor set model
The hardware is all very different so vulkan solution was the descriptor set layout. With that descriptor set layout, the we get the bindings.

For the fix hardware set up, it's pretty straight forward you set up a descriptor set layout and we map the fixed hardware slots to the hardware. With fixed bindings they are typically stored on the CPU-side and the actualy beindings get set as part of the **vkCmdBindDescriptorSets()** or **vkCmdDraw/Dispatch()** . For the other three these are allocated on the GPU memory. For heap descriptors the allocation may be just part of the descriptor set itself in the image or buffer view object to save memory.  

UBOs are weird and complicate things. They have fied hardware bindings even on bindless hardware. This is because UBOs are the hottest of the hot paths and even small differences in UBO fetching can seriously slow things down. There a lot of complexity with this for example on Linux intel has different code paths the UBOs follow depending if they are doing update-after-bind (bindless), which shader stage you're in, what the other UBOs are doing. 

Dynamic buffer act like fixed hardware bindings. How these are implemented changes depending on the driver and hardware. These often have fixed hardware bindings or the descriptor loads things like it's a push constants. Even if the buffer point comes from descriptor set memory the dynamic offset has to get loaded via some push constant thing.
##### The D3D12 descriptor heap
One things that microsoft did really well was with it's D3D12 was to embrace descriptor heaps for their hardware and drivers. In the early days these had to be bound before you can execute any 3D work or compute commands. Shaders have the usual HLSL register notion for describing the descriptor interface. When the shader are compiled you get the descriptor tables used to map the bindings in the shader to ranges in the relevent descriptor heap! While the heap range itself is a fixed the application has access to move the range around at will. 

With SM6.6 in D3D12 Microsoft significantly improve flexibility and further embraced heaps. Instead of using this descriptor table in the root descriptor, applications can use heap indices directly! This provide access to the full bindless experience! The developers just have to worry about managing the heap allocation with the resources lifetimes and figure out how to get the indices into their shaders. Gone of the days of having to fiddle with the interface layouts because you wanted to change 1 things in a shader, now you're entire descriptor layout has to change. Developers loved this change honestly.

A descriptor heap is just a very restrictive buffer. Meaning for AMD drivers just use of their debufer buffer bindings (resource and sampler heaps are seperate in D3D12) and implements a heap as a descriptor buffer.

There is a downside to descriptor heap approach is that is forces some amount of extra indirection, especially with the SM6.6 bindless mdoel. First thing you need to do is load a heap index into a constant buffer and pass that to loadOp/storeOp. This allows the GPU to do the sequeence instruction that fetches the heap, after that it does some offset calculations and does the actual load and store operations for that corrisponding pointer. This can actually slow things down if there are a lot of operations happening, the compiler can remove some but it's not a given that will happen if you have a tonne of stuff.

... Skipping some things because it's getting into the weeds.
###### Towards a better future
