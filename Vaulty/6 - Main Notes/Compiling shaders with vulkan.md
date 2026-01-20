2025-10-19 18:31
Status: #baby 
Tags: [[vulkan]] [[vulkan shaders]]
# Compiling shaders with vulkan

Khronos SDK has a **vender-independent** compiler. The compiler checks your code is correct and when it's compiled you can **ship that code** with your game. This is the CORRECT thing to do.

If you use the glslc.exe compiler it fits closer with the paramter formats with GCC and Clang, not only that you get `#includes` for free. These are both with the vulkan SDK.

A simple way to compile the shader be like this. 
```batch
C:/VulkanSDK/1.4.321.1/Bin/glslc.exe mainShader.vert -o mainShader.vert.spv
C:/VulkanSDK/1.4.321.1/Bin/glslc.exe mainShader.frag -o mainShader.frag.spv
pause
```

The option -o means output file name if successful compiled. glslc.exe will work out the file type with it's file extension.
# References
##### Main Notes
[[Reading in compiled shaders]]
#### Source Notes
[[Drawing a triangle]]