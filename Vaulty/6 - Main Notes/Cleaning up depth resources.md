2025-12-06 19:30
Status: #baby 
Tags: [[vulkan]] [[depth testing]]
# Cleaning up depth resources

Just like with colour images you'll need to clean up resouces 
```c++
vkDestroyImageView(device, depthImageView, nullptr);
vkDestroyImage(device, depthImage, nullptr);
vkFreeMemory(device, depthImageMemory, nullptr);
```
# References
##### Main Notes
[[Creating the depth image and view]]
#### Source Notes
[[Depth buffering]]