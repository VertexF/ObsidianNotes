# Reference Master Graphics Programming with Vulkan

To begin if you want to disassemble SPIR-V code you can use the command line tool that's part of the Vulkan SDK `spirv-dis` This will output text to the terminal. 

Based on this shader code and these command listed below we will discuss the OpCode that comes out of disassembling the SPIR-V code.

1) `C:\VulkanSDK\1.4.321.1\Bin\glslangValidator.exe NotesOnSPIRVShader.vert --target-env vulkan1.3 -o NotesOnSPIRVShaderGLSLVal.spv -S vert`
2) `C:\VulkanSDK\1.4.321.1\Bin\spirv-dis.exe NotesOnSPIRVShaderGLSLVal.spv -o disasmbelyVertShaderGLSLVal.txt`

```c
#version 450

layout (std140, set = 0, binding = 0) uniform LocalConstants
{
	mat4 model;
	mat4 viewProjection;
	mat4 modelInverse;
	vec4 eye;
	vec4 light;
};

layout(location = 0) in vec3 position;
layout(location = 1) in vec4 tangent;
layout(location = 2) in vec3 normal;
layout(location = 3) in vec2 texcoord0;

layout(location = 0) out vec2 vTexcoord0;
layout(location = 1) out vec3 vNormal;
layout(location = 2) out vec4 vTangent;
layout(location = 3) out vec4 vPosition;

void main()
{
	gl_Position = viewProjection * model * vec4(position, 1.0);
	
	vPosition = model * vec4(position, 1.0);
	vTexcoord0 = texcoord0;
	vNormal = mat3(modelInverse) * normal;
	vTangent = tangent;
}
```


Quick note on compilers, glslc is just a wrapper around glslValidator. The OpCode is much easier to read when you use glslValidator and for consistency when doing SPIR-V reflect I recommend you pick a compiler and learn how that works and stick with it. Since I know glslValidator OpCode better I will only be discussing that.

Let start from the top of the **disasmbelyVertShader.txt** which contains all the OpCode in it.

```c
1; SPIR-V
2; Version: 1.6
3; Generator: Khronos Glslang Reference Front End; 11
4; Bound: 74
5; Schema: 0
6               OpCapability Shader
7          %1 = OpExtInstImport "GLSL.std.450"
8               OpMemoryModel Logical GLSL450
9               OpEntryPoint Vertex %main "main" %_ %__0 %position %vPosition %vTexcoord0 %texcoord0 %vNormal %normal %vTangent %tangent
10               OpSource GLSL 450
11               OpName %main "main"
```

Lines 1 to 5 are just things telling you information about the compiler used. Lines 6 to 8 are telling us what GLSL version we used.  Line 9 references the main function with the list of input and output variables. Anything with a % is a variable. It's possible to forward declare a variable which we can use later. 

```c
12               OpName %gl_PerVertex "gl_PerVertex"
13               OpMemberName %gl_PerVertex 0 "gl_Position"
14               OpMemberName %gl_PerVertex 1 "gl_PointSize"
15               OpMemberName %gl_PerVertex 2 "gl_ClipDistance"
16               OpMemberName %gl_PerVertex 3 "gl_CullDistance"
17               OpName %_ ""
```

These lines of code used to tell us what variables are available in the shader. These are injected into the shader. 
# EXPLAIN WHAT %_ "" MEANS LATER.

```c
18               OpName %LocalConstants "LocalConstants"
19               OpMemberName %LocalConstants 0 "model"
20               OpMemberName %LocalConstants 1 "viewProjection"
21               OpMemberName %LocalConstants 2 "modelInverse"
22               OpMemberName %LocalConstants 3 "eye"
23               OpMemberName %LocalConstants 4 "light"
24               OpName %__0 ""
```

Here we have the local constants uniform buffer and their positions within in the struct. Line 24 is 
# EXPLAIN LINE 24

