# Reference https://learnopengl.com/PBR/IBL/Diffuse-irradiance

##### What is Image based lighting
IBL or image based lighting is a collection of techniques that deal with lighting outside of point light and ray tracing. It treats the environment like a big light source. We are basically taking our cube map texture and using each texel as a light emitter. Which makes things look brighter and like they belong to the environment.

So IBL is typically more accurate than even some global illumination stuff and especially ambience. 
##### The goal with using IBL and PBR
We can start to introduce the PBR with IBL if we look at the PBR equation:
$$L_o(p,\omega_o) = \int\limits_{\Omega} 
    	(k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})
    	L_i(p,\omega_i) n \cdot \omega_i  d\omega_i$$

So the goal here is the do the integral over all lighting direction $\omega_i$ over the hemisphere $\Omega$. Because we need to consider every light direction solving the integral is non-trivial, unlike when you know all the direction the light is coming in.

We need to do these two task to simplify our calculations.
1) First task is to grab all the scene radiance given any direction vector $\omega_i$
2) It needs to be fast.
##### Simplifying the PBR equation for diffuse
If you already have a processed cubemap this gets easier to work out. We can visualise every texel of cubemap as one single emitting light source. Based on the PBR equation we can sample over a cubemap and get our the scene radiance based from that direction.

You want things to be fast but we have a problem you'll need to $\omega_i$ over the entire hemisphere $\Omega$ which is far too expensive to do for each fragment shader invocation. To help with speed you want to pre-compute most of the computation. You can split the diffuse $k_d$ and specular $k_s$ in terms of BRDF can be made independent from each other.

$$L_o(p,\omega_o) = 
		\int\limits_{\Omega} (k_d\frac{c}{\pi}) L_i(p,\omega_i) n \cdot \omega_i  d\omega_i
		+ 
		\int\limits_{\Omega} (k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})
			L_i(p,\omega_i) n \cdot \omega_i  d\omega_i$$
If you split of the diffuse and specular parts of the integral you can focus on the diffuse part of the BRDF.

If you look at the lambert term which is a constant term (the colour $c$ the refaction ration $k_d$ and $\pi$) these are not dependent on the integral variable. So you can pull that out with.

$$L_o(p,\omega_o) = 
		k_d\frac{c}{\pi} \int\limits_{\Omega} L_i(p,\omega_i) n \cdot \omega_i  d\omega_i$$
This meant that the integral it's only depent on $\omega_i$ (assuming that $p$ is at the centre of the environment map). With this knowledge, we can calculate or pre-compute a new cubemap that stores in each sample direction (or texel) with $\omega_0$ the diffuse integral's result by convolution. 

Convolution in maths measn we are taking 2 function that produces a third as a integral project of two functions $f*g$ if both f and g are functions.

Here what we are doing is considering all the data in a solution set and applying some kind of computation to every piece data in that same set. In our case the set is the scene's radiance or environment map. Meaning when we are considering a single sample from the cubemap we have to consider all other samples that get sampled over the hemasphere.
##### Making the irradiance map
We'll want to do the convolution by solving the integral for each output $\omega_0$ sample direction. We do this by grabbing a bunch of $\omega_i$ and work them out and sample the average, over the hemisphere $\Omega$ . The hemisphere we need to build up is a bunch of sample in the direction of $\omega_i$ and it oriented towards the output sample of $\omega_0$ direction we are convoluting.

![[ibl_hemisphere_sample.png|348]]
For each sampled direction $\omega_0$ store the result of the diffuse integral. This is a pre-computed sum of all the indirect diffuse light hitting a surface aligned along the direction $\omega_0$ When you work this out you have a irradiance map. This is because we have a convoluted cubemap which effiectly gives us a sample of the scene's irradiance from any direction $\omega_0$

If you're indoors it's going to be common if you have 1 irradance map at point $p$ you're diffuse light calculation is going to break down. How engines deal with this is to have reflection at different point and they caculated there own irradance map and then you do linear interpretation between each probe.

This is what an irradance map looks like based using this cubemap![[ibl_irradiance.png]]

If we store the convoluted result in each of our cubemap texels (in the direction of $\omega_0$) is kind of like an average of the light and colour. This give us our scene irradiance.
##### Linear lighting vs High Dynmanic Range rendering.
If you are using PBR all it's inputs needs to be linear, when you are lighthing the scene in linear space. This means you need to do gamma correctiont to get your light correct.

So when you're doing PBR you want your inputs to be as accurate to the real as possible. Meaning that when you would our the colour for your diffuse `materialColour`

```c
vec3 materialColour = intensity * mix(fresnelMix, conductorFresnel, metalness) * NdotL;
  
materialColour = emissive + mix(materialColour, materialColour * ao, occlusionFactor);
```

Can grow rapidly in LDR (low dynamic range) which means that it can get clamped to `0.0` or `1.0` 

So there are two ways to fix this. 
1) Taking HDR data and correct converting back to LDR
2) Tonemapping it to LDR

These things needs to be done before gamma correction. You can simplify apply a Reinhard operator at the end of your fragment shader to correct the colours in your PBR solution.

The Reinhard operator works way better when you have some ambient lighthing in your scene. This also means you want to do any tonemapping in HDR not LDR otherwise things look grey.

If you've done either just a conversion to LDR from HDR or a tonemap (or both tonemap -> HDR to LDR) then the gamma correction can be done correctly. Meaning that your colour output will be correct for high and low light intensities (in theory). 
##### The problem with LDR cubemaps in PBR
So it is very important for PBR that you deal with HDR properly both for colour [[Linear lighting vs High Dynmanic Range rendering]] and your vulkan swapchain is in the colour space format you expect [[Swapchain colour space format]]. This is because the PBR taking in real physcially properties. Without working in HDR you can't work out the relative intensity of something like a small lightblub and the sun for example.

So for image based lighthing is based on the environment's indirect light intensity on the colour values of a cubemap we need some way to store the lighting's high dynamic range into a the cube.

Remember that when we take in physical inputs for PBR. We need them to be in HDR to accurately describe the relative intensity of lights. If you use a regular image for your cube, you're only going to have the intensity that does from `0.0` and `1.0`.  Meaning for PBR you need the cubemap to be in HDR not LDR. 
##### Swapchain colour space format.
When you are dealing HDR/LDR images when your doing something kind of lighthing in a fragment like PBR. You to be careful that the swapchain image format isn't already coverting your image to linear space, as you can do this twice be accident making everything look grey.

At a very basic level if you're going to covert the colour space yourself you need to set the swapchain image format to be a `UNORM` version of colour space. If you're not doing the coversion you need to set this `SRBG`.

```c++
VkSwapchainCreateInfoKHR swapchainCreateInfo{};
swapchainCreateInfo.sType = VK_STRUCTURE_TYPE_SWAPCHAIN_CREATE_INFO_KHR;
swapchainCreateInfo.imageFormat = 
VK_FORMAT_B8G8R8A8_UNORM; //For non-linear space
VK_FORMAT_B8G8R8A8_SRGB; //For linear space conversion

vkCreateSwapchainKHR(vulkanDevice, &swapchainCreateInfo, nullptr, &vulkanSwapchain);
```

This colour imageFormat is how the image will be interpreted by the presentation engine. The conversion happens while reading/writing.


##### The radianace HDR file format
The radiance file format is a format that store the a cubemap as 6 faces as floating pointer data. So allows us to have the correct light intensity of a given cubemap, because we are not bound by the limited LDR of `0.0` and `1.0`.