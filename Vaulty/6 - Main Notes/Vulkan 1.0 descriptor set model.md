2026-03-08 10:04
Status: #baby 
Tags: [[vulkan]] [[GPU hardware]] [[vulkan descriptors]]
# Vulkan 1.0 descriptor set model

The hardware is all very different so vulkan solution was the descriptor set layout. With that descriptor set layout, the we get the bindings.

For the fix hardware set up, it's pretty straight forward you set up a descriptor set layout and we map the fixed hardware slots to the hardware. With fixed bindings they are typically stored on the CPU-side and the actualy beindings get set as part of the **vkCmdBindDescriptorSets()** or **vkCmdDraw/Dispatch()** . For the other three these are allocated on the GPU memory. For heap descriptors the allocation may be just part of the descriptor set itself in the image or buffer view object to save memory.  

UBOs are weird and complicate things. They have fixed hardware bindings even on bindless hardware. This is because UBOs are the hottest of the hot paths and even small differences in UBO fetching can seriously slow things down. There a lot of complexity with this for example on Linux intel has different code paths the UBOs follow depending if they are doing update-after-bind (bindless), which shader stage you're in, what the other UBOs are doing. 

Dynamic buffer act like fixed hardware bindings. How these are implemented changes depending on the driver and hardware. These often have fixed hardware bindings or the descriptor loads things like it's a push constants. Even if the buffer point comes from descriptor set memory the dynamic offset has to get loaded via some push constant thing.
# References
##### Main Notes
[[The problem with descriptor heaps]]
[[The GPU hardware of descriptors]]
#### Source Notes
[[Descriptors are hard]]
