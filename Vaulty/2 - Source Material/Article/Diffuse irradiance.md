# Reference https://learnopengl.com/PBR/IBL/Diffuse-irradiance

IBL or image based lighting is a collection of techniques that deal with lighting outside of point light and ray tracing. It treats the environment like a big light. We are basically taking our cube map texture and using each texel as a light emitter. Makes things look brighter and like they belong to the environment.

So IBL is typically more accurate than even some global illumination stuff and especially ambient. 

We can start to introduce the PBR with IBL if we look at the PBR equation:
$$L_o(p,\omega_o) = \int\limits_{\Omega} 
    	(k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})
    	L_i(p,\omega_i) n \cdot \omega_i  d\omega_i$$

So the goal here is the do the integral over all lighting direction $\omega_i$ over the hemisphere $\Omega$. Because we need to consider every light direction solving the integral is non-trivial, unlike when you know all the direction the light is coming.

We need to do these two task to simplify our calculations.
1) First task is to grab all the scene radiance given any direction vector $\omega_i$
2) It needs to be fast.

If you already have a process cubemap this gets easier to work out. We can visualise every texel of cubemap as one single emitting light source. Based on the PBR equation we can sample over a cubemap and get our the scene radiance based from that direction.

You want things to be fast but we have a problem you'll need to $\omega_i$ over the entire hemisphere $\Omega$ which is far too expensive to do for each fragment shader invocation. To help with speed you want to precompute or pre-compute most of the computation. You can split the diffuse $k_d$ and specular $k_s$ in terms of BRDF can be made independent from each other.

$$L_o(p,\omega_o) = 
		\int\limits_{\Omega} (k_d\frac{c}{\pi}) L_i(p,\omega_i) n \cdot \omega_i  d\omega_i
		+ 
		\int\limits_{\Omega} (k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})
			L_i(p,\omega_i) n \cdot \omega_i  d\omega_i$$
If you split of the diffuse and specular parts of the integral you can focus on the diffuse part of the BRDF.

If you look at the lambert term which is a constant term (the colour $c$ the refaction ration $k_d$ and $\pi$) these are not dependent on any integral variable. So you can pull that out with.

$$L_o(p,\omega_o) = 
		k_d\frac{c}{\pi} \int\limits_{\Omega} L_i(p,\omega_i) n \cdot \omega_i  d\omega_i$$
This meant that the integral it's only depent on $\omega_i$ (assuming that $p$ is at the centre of the environment map). With this knowledge, we can calculate or pre-compute a new cubemap that stores in each sample direction (or texel) with $\omega_0$ the diffuse integral's result by convolution. 

Convolution in maths measn we are taking 2 function that produces a third as a integral project of two functions $f*g$ if both f and g are functions.

Here what we are doing is considered all the data in a solution set and applying some kind of computation that to every piece data in that same set. In our case the set the scene's radiance or environment map. Meaning when we are considering the a single sample from the cubemap we have to consider all other samples that get sampled over the hemasphere.

We'll want to do the convolution by solving the integral for each output $\omega_0$ sample direction. We do this by grabbing a bunch of $\omega_i$ and work them out and sample the average, over the hemisphere $\Omega$ . The hemisphere we need to build up is samples the direction of $\omega_i$ and it oriented towards the output sample of $\omega_0$ direction we are convoluting.

![[ibl_hemisphere_sample.png]]

