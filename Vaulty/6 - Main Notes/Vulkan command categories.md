2026-04-27 12:13
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]] [[vulkan command buffers]]
# Vulkan command categories

When we are talk about synchronisation we are concerned with **Actions commands** and what order they execute on the GPU. We use **Synchronisation commands** to do that. **State commands** are not effected by by synchronisation, as they change the state of the command buffer itself while recording, binding and setting up dynamic parts of the pipeline.  

Here is a a short list of the different commands and there category.

>**Action commands**
 Graphics Pipeline Commands:  
>>vkCmdDraw
>>vkCmdIndexed
>>vkCmdIndirect
>>vkCmdDrawMeshTasksNV
>>vkCmdClearAttachments
 Compute Pipeline Commands:
>>vkCmdDispatch
>>vkCmdDispatchIndirect
>>Ray Tracing Pipeline Commands:
>>vkCmdTraceRaysKHR
>>vkCmdTraceRaysIndirectKHR
 Transfer Commands:
>>vkCmdCopyBuffer
>>vkCmdCopyImage
>>vkCmdCopyBufferToImage
>>vkCmdCopyImageToBuffer
>>vkCmdCopyAccelerationStructureKHR
>>vkCmdBlitImage
>>vkCmdResolveImage
>>vkCmdClearColorImage

Each of the subcategories in this list use different queues to get the work done. 

>**Sychronisation commands**
>>vkCmdPipelineBarrier/2
>>vkCmdSetEvents/2
>>vkCmdWaitEvents/2
>>vkCmdBeginRenderPass/2
>>vkCmdEndRenderPass/2

>**State commands**
 Bind Commands:
>>vkCmdBindDescriptorSets
>>vkCmdBindPipeline
>>vkCmdBindVertexBuffers
>>vkCmdBindIndexBuffer
 Other Commands:
>>vkCmdPushConstants
>>vkCmdPushDescriptorSetKHR
>>vkCmdSetScissor
>>vkCmdSetViewport
>>vkCmdSetDepthBias
# References
##### Main Notes
[[What are semaphores]]
[[What are fences]]
#### Source Notes
[[Vulkan synchronisation explained]]