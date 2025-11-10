### Overview
[[What is Vulkan]]
[[Code for the simplest vulkan app]]
### Instance
[[First Step for setting up Vulkan]]
[[Creating a vulkan instance]]
[[Setting up extensions]]
[[Checking for supported extensions]]
[[Running Vulkan on Mac]]
[[Cleaning a Vulkan Instance]]
### Useful to know
[[Loading extended functions dynamically]]
[[Dealing with VK_EXTENSION_NOT_PRESENT error]]
### Validation
[[What are validation layers]]
[[Vulkan Layers aren't extensions]]
[[Instance and device level validation]]
[[Enabling Validation layers]]
[[Why set up debug callbacks for vulkan validation]]
[[Creating the custom callback function for validation]]
[[Connecting custom debug function to Vulkan]]
[[How to debug instance creation]]
[[Customising validation layers externally]]
### Physical Device
[[What is a vulkan physical device]]
[[Listing the GPUs in Vulkan]]
[[Checking for hardware suitability]]
### Queue Families
[[What are queue families]]
[[Listing the queue families]]
[[Checking for family queue suitability]]
[[Cleaning up a physical device and queue families]]
### Logical Device and Queues
[[What is vulkan logical device]]
[[Setting up vulkan queues]]
[[Device level extensions]]
[[Making GPU features usable]]
[[Create the logical device]]
[[Cleaning up a vulkan logical device and queues]]
[[Creating vulkan queues]]
[[Creating a separate queue]]
### Vulkan surface
[[What is a vulkan surface]]
[[Vulkan surface extensions]]
[[Creating a vulkan surface]]
[[Destroying a vulkan surface]]
[[Checking to see if presentation is supported]]
### Swapchain
[[What is a swapchain]]
[[Checking surface and swapchain compability]]
[[Setting up the swapchain extension]]
[[Querying surface capabilities]]
[[Checking surface format]]
[[Selecting a presentation mode]]
[[Creating the swapchain]]
[[Getting swapchain images]]
>**Swapchain Recreate**
>[[Recreating the swapchain]]
>[[When to recreate the swapchain image]]
>[[Fixing a possible deadlock]]
>[[Dealing with VK_ERROR_OUT_OF_DATE not being supported]]

### Image View
[[What are image views]]
[[Creating an image view]]
[[Destroying a image view]]
### Graphics Pipeline
[[What is the graphics pipeline]]
>**Shaders**
>[[Compiling shaders with vulkan]]
>[[Reading in compiled shaders]]
>[[Creation of the shader module]]
>**Fixed Functions** 
>[[Setting up the dynamic states]]
>[[Setting up the vertex inputs]]
>[[Setting up the input assembly]]
>[[Setting up the viewports and scissors]]
>[[Setting up the rasteriser]]
>[[Setting up the multisampler]]
>[[Setting up the depth and stencil testing]]
>[[Setting up the colour blending]]
>**Pipeline Layout**
>[[Setting up the pipeline layout]]
>**Render pass**
>[[What is a render pass]]
>[[Setting up the colour attachment descriptions]]
>[[Setting up the attachment reference]]
>[[Setting up the subpass]]
>[[Setting up the render pass]]
>**Graphics Pipeline Creation**
>[[Creating the graphics pipeline]]
>[[Creating derived graphic pipelines]]

### Subpass
[[What is a subpass]]
[[What are subpass dependencies]]
[[Setting up subpass dependencies with one subpass]]
### Framebuffers
[[What is a framebuffer]]
[[Creating the framebuffers]]

[[Handling frames in flight]]
### Command Buffers
[[What are command buffers]]
[[Setting up the command pool]]
[[Setting up the command buffers]]
[[Recording commands]]
[[Using a render pass]]
[[Setting up and running the vkCmdDraw command]]
### Synchronisation
[[What is synchronisation in vulkan]]
[[What are semaphores]]
[[What are fences]]
[[What you need to synchronise a frame]]
[[Setting up the synchronisation objects]]
[[Using a fence to synchronise the previous frame]]
[[Acquirng an image from the swapchain]]
[[Submitting the command buffer for rendering]]
[[Setting the synchronisation with the subpass dependencies]]
[[Presenting to the screne]]