# Reference https://vulkan-tutorial.com/Vertex_buffers/Vertex_input_description

##### Vertex input description
If you want to use a shader that gets data uploaded to the GPU to be processed. To process shader vertex attributes that take in vertex data as input in the shader and do something you with them you need to create inputs for the vertex shader. The simplest set up would look something like this

```c++
layout (location = 0) in vec2 inPosition;
layout (location = 1) in vec3 inColour; 

layout(location = 0) out vec3 fragColour;

void main()
{
	gl_Position = vec4(inPosition, 0.0, 1.0);
	fragColour = inColour;
}
```

This takes in both position and colour as inputs and uses them in the shader. Remember this shader runs per vertex.

Not all types take up simply 1 slot in the shader. The x64 version of vec3 called dvec3 actually take up 2 slots. If you need something like this look at the OpenGL layout qualifers https://wikis.khronos.org/opengl/Layout_Qualifier_(GLSL).
##### Vertex data
There isn't much to be said here about vertex data. This could be just position or other stuff. For this tutorial we are creating colour and position with a Vertex struct that contains these data types.

This will eventually come from models so this isn't important really.