2026-03-08 10:00
Status: #baby 
Tags: [[vulkan]] [[GPU hardware]] [[vulkan descriptors]]
# The GPU hardware of descriptors

To understand problem of descriptors you have to realise that Vulkan is targeting a tonne of hardware. So lets dig into the hardware.

There are 4 possible descriptor set implementations:

1) **Direct Access (D)** This is were the shader itself passes the entire thing over to the GPU to be access directly. 
2) **Descriptor Buffers (B)** The hardware has a piece of memory were a buffer lives. To access the descriptor the GPU needs to read that piece of memory to access the data. Meaning that instead of passing the entire descriptor, it lives in the buffer now. To access were the buffer lives either on a fixed address or the base address is passed to the shader.
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
**Intel (pre-Skylake)** is little more flexible than this table makes out. It uses a heap like structure but has a level of indirection that is limited to 240. **Post-skylake** it uses a different set up but still has the heap thing it allows a backdoor to do **Descriptor Heap** stuff.
# References
##### Main Notes
[[Vulkan 1.0 descriptor set model]]
[[The problem with descriptor]]
#### Source Notes
[[Descriptors are hard]]