2025-09-21 08:06
Status: #adult 
Tags: [[vulkan]] [[vulkan validation]]
# Instance and device level validation

In the early days of vulkan, khronos split up validation into instance level which was global, and device level validation which was meant to track things that were GPU specific so anything that used the vulkan logical device.

Device level validation has been depreciated for awhile now and only instance level validation is used. However, it's not hard to turn on device level validation and doesn't cause any issue if it's not used. So it's recommend you turn it on anyway just in you come into contact with an older GPU.
# References
##### Main Notes
[[What are validation layers]]
[[How to debug instance creation]]
#### Source Notes
[[Vulkan-Tutorial]]