1) `VK_EXT_host_image_copy` is an extension that allows you to upload images without them being visible. I believe it's not yet super widely support but it's worth looking up on vulkan website that checks how well supported an extension.

	This is tied into with the 2026 roadmap with vulkan to have better access to memory. The end goal is to have the entire GPU VRAM to have no size limits and be completely host visible.

	This is support completely if you're motherboard has rebar (Dive into rebar and look into how supported). This all means you don't need staging and can go straight to upload the data directly.

2) Start to look into ktx, so you can handle textures better. Note your cookbook has away to convert these.

3) Apparently IBL is an older technique which is used by older games to do something that physical based atmosphere and global illumination does. Look up PBAM (physically based atmospheric model) and SUN comes as they can be used for simulate space better than IBL can. 
	1) Physically Based Sky, Atmosphereand Cloud Rendering in Frostbite by S´ebastien Hillaire is something worth looking at.
	2) This information requires you to be pretty good if not basically know PBR inside out to be able to do the more advanced stuff.
	
4) To be able to do more advanced graphics when it come to pure rendering (excluding things like animation, meshlets, culling, etc). You need a strong base in PBR to be able to launch from that position to make things work nicely.
5) What does the engine:
	1) Profiling - IMPORTANT
		1) How useful is the Visual Studio profiler.
		2) Tracy
		3) Perf?
		4) Make sure if Nsight doesn't enough profiling details, add more debug lables. With maybe command debug labels.
	2) GPU based View frustrum culling
		1) Memory barriers
		2) Compute shaders
	3) Instance + indirect drawing
		1) Note this is extremely powerful with view frustum
	4) Threading
		1) TaskFlow.
	5) Deferred rendering/forward+ (Make sure you pick something that actually fits your engine/game)
		1) if Deferred Rendering
		2) Building up the GBuffer.
		3) Making GBuffer small.
	6) Frame graph
		1) Before we start adding different passes for each frame we need to decide and implement the deferred renderer/forward+ So we can have a main pass 
		2) Pre-depth pass is the most important for overdraw.
	7) Texture compression
		1) Do we need supercompression for single 2D textures that don't have a big resolution. I image that it's faster to do a transcode if you haven't already supercompressed image.
	8) Getting gltfpack working. - Do when stuck something.
	9) An LOD system
		1) Meshoptimiser.
	10) Light clustering.
6) What does the game need
7) Physically Based Rendering in Filament md.html is to help with PBR https://google.github.io/filament/main/filament.html It's the filament engine documentation.
	1) Don't worry too much when they say things are "slow" you are targetting GPUs.
	2) You also use full precision in the **BRDF** not half. 
8) Better space rendering https://sebh.github.io/publications/egsr2020.pdf
9) Give a basic roadmap on learning game engine development. - Note add JTags advice in. 
	1) Also make VERY clear that my advice can ONLY take up to were I am and I'm not an expert (I only know what I know). 
10) A good text editor might zed text editor for linux, you will need to set things up.
11) Physically Based Rendering in Filament md.html is to help with PBR https://google.github.io/filament/main/filament.html It's the filament engine documentation.
	1) Don't worry too much when they say things are "slow" you are targetting GPUs.
	2) You also use full precision in the **BRDF** not half. 
12) Better space rendering https://sebh.github.io/publications/egsr2020.pdf
13) Give a basic roadmap on learning game engine development. - Note add JTags advice in. 
	1) Also make VERY clear that my advice can ONLY take up to were I am and I'm not an expert (I only know what I know).

Particles Plan - 
I need to be able redisplay the particles after the age has died. I need to be able to do this by group of particle sets. Effectively I need to re-run a simulation again after it's finished. So I need to run the compute shaders again to re-do the simulation while having things reset.

So I need to reset the particle set compute shaders buffer that handles the particles. The problem is right now is that I need to be able run more than 1 particle simulation at once. How do I do this?

Current issues
 1) We can't run two compute shaders to run a different particle simulation because it's overwriting the old data. How do you add another particle simulation when it's writing to the same buffer? Potential issue with offsets is that when 1 particle simulation has competely finished we keep
 2) Step 2 We need to reduce to global offset into the buffer when stuff no longer on the screen. 
### TT
If you have a fix variable rate of delta it speeds up the simulation when the frames increase.
### Rachit
.exr format are harder to load because of the libraries but they are the HDR images similar to .hdr but higher quality. Meaning if you want to have highest quality HDR image for your IBL you're gonna have .exr. It's possible that tinyEXR is unstable/slow but unverified.
### ScieCode
If you are viewing something close to the camera (or your eye), the area covered by the angular angular resolution is small, so you may use large textures and it won't alias. However if you view something far away, the area for the same angular resolution is large, so light reaching the same pixel would come from different places, so you use mipmap to replicate this effect, where the final visual is an average of the thing you are looking at. It's why mipmaps are a thing
### Demitry
Checking out vectorisation is cool C++ stack feature to check out.