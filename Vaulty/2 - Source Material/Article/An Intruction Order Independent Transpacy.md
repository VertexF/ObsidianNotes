# Reference https://learnopengl.com/Guest-Articles/2020/OIT/Introduction

##### Exact OIT
The sorting stage requires relatively large amounts of temporary memory in shaders that is usually conservatively allocated at a maximum, which impacts memory occupancy and performance. 

So to solve this we need to fully ulitize SIMT (single instruction mulitple threads) to reduce throughput operation latency. I believe this is due to the fact that when you increase the amount of warps being used you increase the amount of visible memory for the OIT process.

Using [BMA](http://diglib.eg.org/handle/10.2312/PE.PG.PG2013short.059-064) (backwards memory allocation) you can group pixels by their depth complexity and sort them in batches. This would similar to something like [external merge sort](https://en.wikipedia.org/wiki/External_sorting#External_merge_sort) which tries to use the GPU's memory hierarchy and sorting in register. (Learn later).
###### Approximate OIT
This is all about partly sorting things and not storing everything for faster results.

Weighted, blended is a technique that uses a weighting function and two buffer for pixel colour and pixel reveal threshold for the final composition pass. 

You typically deals with things in mulitple passes. Meaning that you will need to set up the pipeline barries for different passes for OIT stuff.
1) First you draw all the solid object that are opaque.
2) Next you draw all the translucent objects.
3) Third you composite the images from both step 1 and 2 and draw them onto a backbuffer (is this a Gbuffer? What's backbuffer).

This apparently is used across all different techniques.
### Part 2 Weigted Blended
# Reference https://learnopengl.com/Guest-Articles/2020/OIT/Weighted-Blended

This attempts to avoid the cost of storing and sorting primitives or fragments. It does this by altering the compositing operationg (what is that?) so that it is order indepent, thus allowing a pure streaming approach
###### The theory of weight blended techniques
This technique renders non-refractive, monochrome transmission through surfaces that themselves have colour, this doesn't require sorting or hardware.  Apparently this can be worked out in a one shader for any GPU that support blending to render targets with more than 8-bits per channel.

When we are rendering to a lower resolution image the cost memory cost remains the same but the bandwidth cost is proportional to the resolution.
##### IMPORTANT
The key idea with weighted, Blended is that we first computes the coverage of the background by transparent surfaces exactly, but only approximate the light scattered towards the camera by the transparent surfaces themselves.

The algorithm imposes a heuristic on inter-occlusion factor amoung transparent surfaces that increases with distance from the camera. Inter-occlusion is something that happens when a transparent object partly occludes light. The heuristic is the weighted function. 

Next step is after all the transparent surfaces have been rendered, we then need to performance a normalised fullscreen compositing pass to reduce errors where the heuristic was a poor approximation for true inter-occlusion.

So you can look up the original on [Weight based OIT](file:///home/user/Downloads/McGuire2013Transparency.pdf) of page 5, through 7 to understand the details of the weighted function. However, there have been improvements to the algorithm over years.
##### Limitation
The biggest limitation is the the heuristic must be tuned for both the depth range and opacity of transparent surfaces. 

So there are 3 different implements of this weighted blended approach which gives a good set of weight functions the paper (original?) tells us how to spot bad weighted blended functions and what to do fix them.

It might be a struggle to implement deferred rendering, since when you apply the lighting pass you are overwriting pixels and this means you can lose information about the previous layers (??) this all means we can't correctly accumulate the colours for the lighting stage.