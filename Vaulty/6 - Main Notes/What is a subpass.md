2025-11-07 14:12
Status: #baby 
Tags: [[vulkan]] [[vulkan subpass]]
# What is a subpass

A subpass is part of the render pass that sets up the dependencies and the attachments between the subpasses. 

These attachments are access attachments which tells the vulkan were you want things to happen for a given subpass. These are the render targets for the subpass.

The subpass actually read and writes to these attachments, this is why you need to set up the dependencies between different subpass so they can read the correct data or write the data need to pass over to the next subpass in the attachment. The reading and writing operations happens at different stages of the graphics pipeline so you need to wait on them too as part of the dependencies of these subpasses. 

So for example in 1 render pass, you might want to write to a depth buffer with on one subpass and on another that writes to a colour buffer in a different subpass. These two subpass performance operations at different stages of the graphics pipeline onto different attachments. 

Note these attachments are just access attachments, which isn't the same as the attachments for loading and storing data you did when [[Setting up the colour attachment descriptions]] for the render pass itself.
# References
##### Main Notes
[[What are subpass dependencies]]
[[What is a render pass]]
[[Setting up the colour attachment descriptions]] 
#### Source Notes
[[Render Passes in Vulkan]]
