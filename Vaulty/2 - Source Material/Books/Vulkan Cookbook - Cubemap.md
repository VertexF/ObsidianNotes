# Reference Vulkan 3D Graphics Rendering Cookbook Implementation
page 116

A cube map is a textures that is made of 6 invidual 2D textures. A cube map can be easily sampled from sampled from a directional vector. This can come in useful with physical based rendering because you can store a irradiance cube map.

Typically you have a equirectangular projections these are projected longitude and latitude vertical and horizontal lines to straight lines, make it an easy and popular way to store light probe images.
