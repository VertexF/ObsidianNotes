2025-11-07 14:12
Status: #baby 
Tags: [[vulkan]] [[vulkan subpass]]
# What is a subpass

A subpass is part of the render pass that sets up the dependencies with attachments between the subpasses and itself. Command buffers are recorded into a subpass when using a render pass instance. Meaning that actual operations from the command buffer get executed here. 

So think about a subpass that's something that runs commands in some order, they have inputs and outputs, and there are dependencies between themselves and other subpasses.

Remember we are **NOT** actually writing any data to a subpass, they are interested in ordering graphics command and other operations.

So each part of the graphics pipeline has an input and and output that feeds to the next part of the pipeline. When we are talking about **VkSubpassDescription** we are talking about the inputs and outputs of different parts of the graphics pipeline, were different GPU operations are taking place. [[Setting up the colour attachment descriptions]] 

This is important to understand because with attachment references **VkAttachmentReference** we will tell the subpass what the output layout location within the fragment shader is. This means that the subpass knows what the output of the fragment shader stage is. [[Setting up the attachment reference]]

The attachments for dependencies in **VkSubpassDependency** force the GPU to finish operations before a particular point. For example presenting an image is an operation that is dependent on the output of the colour blending operations. That is what the attachment dependencies are doing within a subpass.
 
The attachments to build dependencies are either graphics pipeline stage attachmnts such as **VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT** or they are access attachments such as **VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT**, both are just enums. These attachments tell vulkan that we want to wait for operations **AND** the read or write of the outputs to finish before continuing within a subpass.

You need to set up the dependencies between different subpasses so they can read the correct data or write the data needed to pass over to the next subpass at the correct time.

This access dependencies wait to for the read or write operation to be completed **after** the graphics command has finished.

So for example in 1 render pass, you might want to write to a depth buffer with on one subpass and with another subpass you might want to write to a colour buffer. These two subpass performance operations at different stages of the graphics pipeline. So we need to set up dependencies because in this case the depth buffer needs to have written data out for the colour buffer to use. 
# References
##### Main Notes
[[What are subpass dependencies]]
[[What is a render pass]]
[[Setting up the colour attachment descriptions]] 
#### Source Notes
[[Render Passes in Vulkan]]
