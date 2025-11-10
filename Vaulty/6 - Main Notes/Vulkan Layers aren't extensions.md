2025-09-21 08:11
Status: #adult  
Tags: [[vulkan]] [[vulkan validation]]
# Vulkan Layers aren't extensions

Layers and extensions are set up pretty much in the same way, however it's important to not that extensions and layers do different things.

Extensions extend vulkan core functionality, layers are a proxy between vulkan function calls and the driver. This is important because layers have an overhead cost that slows your renderer down, extensions just add features for you to use and don't slow things down but change how things might be used.
# References
##### Main Notes
[[Enabling Validation layers]]
[[Setting up extensions]]
[[What are validation layers]]
#### Source Notes
[[Drawing a triangle]]