# Reference https://vulkan-tutorial.com/

#### Overview
Vulkan is a graphics API from khronos and it's a successor to OpenGL. It's cross-platform works across all the GPUs. 

Back in the day you would just send in the vertex data in a standard format, then the graphics driver takes over the everything including shading lighting. Now graphics venders have slowly updated the hardware and the graphics drivers have slowly opened to fit the programmers intent.

Tiled rendering happens on mobile GPUs and another concern with older graphics APIs is that they aren't really multithreading friend which can bottleneck stuff on the CPU side.

Vulkan unifies compute and graphics work under one system instead of only being for graphics like OpenGL.