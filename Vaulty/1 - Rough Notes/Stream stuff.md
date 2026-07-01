1) Make some notes on placement new

2) `VK_EXT_host_image_copy` is an extension that allows you to upload images without them being visible. I believe it's not yet super widely support but it's worth looking up on vulkan website that checks how well supported an extension.

	This is tied into with the 2026 roadmap with vulkan to have better access to memory. The end goal is to have the entire GPU VRAM to have no size limits and be completely host visible.

	This is support completely if you're motherboard has rebar (Dive into rebar and look into how supported). This all means you don't need staging and can go straight to upload the data directly.

3) Start to look into ktx, so you can handle textures better. Note your cookbook has away to convert these.

4) Apparently IBL is an older technique which is used by older games to do something that physical based atmosphere and global illumination does. Look up PBAM (physically based atmospheric model) and SUN comes as they can be used for simulate space better than IBL can. 
	1) Physically Based Sky, Atmosphereand Cloud Rendering in Frostbite by S´ebastien Hillaire is something worth looking at.
	2) This information requires you to be pretty good if not basically know PBR inside out to be able to do the more advanced stuff.
	
5) To be able to do more advanced graphics when it come to pure rendering (excluding things like animation, meshlets, culling, etc). You need a strong base in PBR to be able to launch from that position to make things work nicely.

6) What does the engine/game need.
	- **More entity interaction.
		- Need particles for point collision detection. - What is the meta for particles/how do you do them with Jolt.
		- The player needs to die/win.
		- Fix players movement, maybe less important that 2D renderer. - Try and fix on Windows first.
	- **2D renderer for UI and other stuff.**
	- Still needs improvement. 
	- I also need to consider how to release this game and stick to it.
		- Release this game, and make another one that's better with better etc.
7) Physically Based Rendering in Filament md.html is to help with PBR https://google.github.io/filament/main/filament.html It's the filament engine documentation.
	1) Don't worry too much when they say things are "slow" you are targetting GPUs.
	2) You also use full precision in the **BRDF** not half. 
8) Better space rendering https://sebh.github.io/publications/egsr2020.pdf
9) Give a basic roadmap on learning game engine development. - Note add JTags advice in. 
	1) Also make VERY clear that my advice can ONLY take up to were I am and I'm not an expert (I only know what I know). 
10) A good text editor might zed text editor for linux, you will need to set things up.