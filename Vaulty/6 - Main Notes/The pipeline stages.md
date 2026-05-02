2026-04-27 13:08
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# The pipeline stages

Action command which are everything other than command buffer state change, synchronisation and indirect commands consist of multiple operations, which go pipeline stages. The pipeline stages that get executed to depend on the action command and the state of the command buffer.

In the pipeline stage they themselves aren't strictly followed by the actual work. This is because some stages can be merged and in the`VK_PIPELINE_STAGE` some stages are left out. The graphics driver will do this and it's not something you have to worry about (in theory).

![[command_pipeline_stages.png]]

You also have three special stages that have special access OR combining stages.
1) `VK_PIPELINE_STAGE_HOST_BIT`
2) `VK_PIPELINE_STAGE_ALL_COMMANDS_BIT`
3) `VK_PIPELINE_STAGE_ALL_GRAPHICS_BIT`

A command you submit goes from `VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT` or `VK_PIPELINE_STAGE_NONE` (**synch2**) to`VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT` or `VK_PIPELINE_STAGE_ALL_COMMANDS_BIT` (**synch2**). So we are adding synchronisation to the pipeline line stages for all commands to order the commands you care about
##### The mysterious TOP_OF_PIPE_BIT and BOTTOM_OF_PIPE_BIT stages. 
`VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT` is actually were the command gets parsed and `VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT` is were the command returns. In **synchronisation2** `TOP_OF_PIPE_BIT` is `NONE` and `BOTTOM_OF_PIPE_BIT` and `ALL_COMMANDS`.
# References
##### Main Notes
[[What are pipeline barriers]]
[[In-queue execution pipeline barriers]]
[[Vulkan command categories]]
[[Implicit synchronisation guarantees]]
[[Basic queue synchronisation]]
[[Vulkan command overlap]]
#### Source Notes
[[Vulkan synchronisation explained]]