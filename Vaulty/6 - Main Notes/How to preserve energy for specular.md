2026-06-18 15:22
Status: #baby 
Tags: [[graphics theory]] [[physically based rendering]]
# How to preserve energy for specular

We can use a white furnace, a uniform lighting environment set to pure white, to validate the energy preservation property of a BRDF. When you do this pure reflective metnals are indistinguishable from the background, no matter the roughness of said surface. 

![[material_furnace_energy_loss.png]]
This should what a surface looks with specular BRDF presented in [[Energy issues with Cook-Torrance BRDF]]  

Mutliple-scatter microfacets BRDFs are discussed in depth in [Heitz2016](https://dl.acm.org/doi/10.1145/2897824.2925943) Sadly this only presents a stochatic evalution of multiscattering BRDF. Therefore this is not suitable for real-time rendering. Kulla and Conty present a different approach in [Kulla17](https://fpsunflower.github.io/ckulla/data/s2017_pbs_imageworks_slides_v2.pdf) Their idea is to add an energy compensation term a an additional BRDF lobe should in this equation.
$$f_{ms}(l,v) = \frac{(1 - E(l)) (1 - E(v)) F_{avg}^2 E_{avg}}{\pi (1 - E_{avg}) (1 - F_{avg}(1 - E_{avg}))}$$
$E$ is where the direction albedo of the specular BRDF $f_r$ with $f_0$ set to 1. Which I believe is the colour of the specular light reflected back.
$$E(l) = \int_{\Omega} f(l,v) (n\cdot v) dv$$
The term $E_{avg}$ is the consine-weighted average of $E$
$$E_{avg} = 2 \int_0^1 E(\mu) \mu d\mu$$
$F_{avg}$ is the cosine-weighted average of the Fresnel term:
$$F_{avg} = 2 \int_0^1 F(\mu) \mu d\mu$$
Both terms $E$ and $E_{avg}$ can be precomputed stored in lookup tables. $F_{avg}$ can simplified when the Schlick approximation
$$F_{avg} = \frac{1 + 20f_{0}}{21}$$
The new lobe is combined $f_r$ with the orignal single scattering lobe, which is the original specular BRDF $f_{ss}$ and the multi-scatter lobe $f_{ms}$ which gives us
$$f_{r}(l,v) = f_{ss}(l,v) + f_{ms}(l,v)$$
So according to Emmanuel Turquin, Lagarde and Golubev makes the observation that the $F_{avg}$ can be just $f_{0}$ with the addtion of applying energy compensation by adding scaled GGX specular
$$f_{ms}(l,v) = f_{0} \frac{1 - E(l)}{E(l)} f_{ss}(l,v)$$
So we of course can pre-compute $E(l)$ via a lookup table it can also be shared with image-based lighting for pre-integration. So multiscattering energy compensations equation is
$$f_r(l,v) = f_{ss}(l,v) + f_{0} \left( \frac{1}{r} - 1 \right) f_{ss}(l,v)$$
Where $r$ is defined as
$$r = \int_{\Omega} D(l,v) V(l,v) \left< n \cdot l\right> dl$$
So to simply the $r$ with this integral we can have DFG lookup table we simplify things to be like this
```c
vec3 energyCompensation = 1.0 + f0 (1.0 / dfg.y - 1.0);
//Scale the specular lob to account for multiscatter.
F *= energyCompensation;
```

#### NOTE: - I have not yet learnt how to precompute DFG lookup tables.

# References
##### Main Notes
[[Energy issues with Cook-Torrance BRDF]]
[[Normal distribution function (specular D)]]
[[Geometric shadowing(specular G)]]
[[Geometric shadowing(specular G) for mobile GPUs]]
[[Fresnel (specular F)]]
#### Source Notes
[[Filament PBR - Materials]]