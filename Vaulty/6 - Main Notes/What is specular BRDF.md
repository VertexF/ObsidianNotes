2026-06-16 15:24
Status: #baby 
Tags: [[graphics theory]] [[physically based rendering]]
# What is specular BRDF

So using the Cook-Torrance approximation of microfacet model integration with with Fresnel law for our BRDF. 
$$f_r(v, l) = \frac{D(h,\alpha)G(v,l,\alpha)F(v,h,f_0)}{4(n \cdot v)(n \cdot l)}$$
We must approximate for the three terms $D, G$ and $F$. If you use [this article](https://graphicrants.blogspot.com/2013/08/specular-brdf-reference.html) you'll be able to see a list of equation that approximate different parts of with the Cook-Torrance specular **BRDF**.
# References
##### Main Notes
[[What is the BRDF]]
[[Introduction to BRDF]]
[[What are dielectrics and conductors]]
#### Source Notes
[[Filament PBR - Materials]]