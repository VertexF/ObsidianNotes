2026-06-08 11:49
Status: #baby 
Tags: [[graphics theory]] [[physically based rendering]] [[image based lighting]]
# The problem with LDR cubemaps in PBR

So it is very important for PBR that you deal with HDR properly both for colour [[Linear lighting vs High Dynmanic Range rendering]] and your vulkan swapchain is in the colour space format you expect [[Swapchain colour space format]]. This is because the PBR taking is taking in real physcially properties. Without working in HDR you can't work out the relative intensity of something like a small lightblub and the sun for example.

So with image based lighthing that's based on the environment's indirect light intensity on the colour values of a cubemap. We need some way to store the lighting's high dynamic range into a the cube.

Remember that when we take in physical inputs for PBR. We need them to be in HDR to accurately describe the relative intensity of lights. If you use a regular image for your cube, you're only going to have the intensity that goes from `0.0` and `1.0`.  Meaning for PBR you need the cubemap to be in HDR not LDR. 
# References
##### Main Notes
[[Linear lighting vs High Dynmanic Range rendering]]
[[Swapchain colour space format]]
#### Source Notes
[[Diffuse irradiance]]