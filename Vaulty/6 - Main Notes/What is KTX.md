2026-07-25 11:03
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]]
# What is KTX

The KTX file format is by Khronos. This is a format that's native to the GPU so there is no need to convert to a different type. It support mip mapping 3D texture and cubemaps. One tool for creating KTX image files is [PVRTexTool](https://developer.imaginationtech.com/solutions/pvrtextool/). This tool allows you to create cubemaps, IBL environment textures and add mip maps to texture.

There are 2 version ktx within the same library, version 1 is getting relatively old here. The two different formats for the actual files themselves are `.ktx` for version 1 and `ktx2` for version 2. If you can, use version 2. This kind all works like Vulkan version 2 of some function. 
# References
##### Main Notes
[[Setting up the KTX library]]
[[Creating a texture with KTX file]]
#### Source Notes
[[How to vulkan]]