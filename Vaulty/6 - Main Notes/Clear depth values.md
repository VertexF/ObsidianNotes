2025-12-06 19:23
Status: #baby 
Tags: [[vulkan]] [[depth testing]]
# Clear depth values

We now have multiple attachment with **VK_ATTACHMENT_LOAD_OP_CLEAR** we also need to specify multple clear values.

```c++
std::array<VkClearValue, 2> clearColour{};
clearColour[0] = { { 0.f, 0.535f, 1.f, 1.f } };
clearColour[1] = { { 1.f, 0.f } };
renderPassBeginInfo.clearValueCount = uint32_t(clearColour.size());
renderPassBeginInfo.pClearValues = clearColour.data();
```

The range for regular perspective depth testing is 0.0 to 1.0 were 1.f lies at the far view plane and 0.f lies at the near view plane. The initial value so the **1.f** should be at the furthest possible depth which is 1.f. For reverse perspective it would be 0.f.
# References
##### Main Notes
[[Setting depth buffer attachments]]
[[Framebuffer with depth]]
#### Source Notes
[[Depth buffering]]