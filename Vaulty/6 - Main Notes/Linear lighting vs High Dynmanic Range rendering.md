2026-06-08 11:38
Status: #baby 
Tags: [[graphics theory]] [[physically based renderer]] [[image based lighting]]
# Linear lighting vs High Dynmanic Range rendering

If you are using PBR all it's inputs needs to be [[What is linear space|linear]], when you are lighing the scene in linear space. This means you need to be careful with how you handle gamma correction to get your lighting correct.

So when you're doing PBR you want your inputs to be as accurate to the real world as possible. Meaning that when you calculate the final colour for your diffuse `materialColour`

```c
vec3 materialColour = intensity * mix(fresnelMix, conductorFresnel, metalness) * NdotL;
  
materialColour = emissive + mix(materialColour, materialColour * ao, occlusionFactor);
```

Things can grow rapidly in LDR (low dynamic range) which means that it can get clamped to `0.0` or `1.0` 

So there are three ways to fix this. 
1) Taking HDR data and correct converting it back to LDR
2) Tonemapping it to LDR
3) Using the vulkan swapchain to automatically convert your images to LDR

These things needs to be done before gamma correction. You can simplify apply a Reinhard operator at the end of your fragment shader to correct the colours in your PBR solution.

The Reinhard operator works way better when you have some ambient lighthing in your scene. This also means you want to do any tonemapping in HDR not LDR otherwise things look grey.

If you've done either just a conversion to LDR from HDR or a tonemap (or both tonemap -> HDR to LDR) then the gamma correction can be done correctly. Meaning that your colour output will be correct for high and low light intensities (in theory). 
# References
##### Main Notes
[[What is linear space]]
#### Source Notes
[[Diffuse irradiance]]
