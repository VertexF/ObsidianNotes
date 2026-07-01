2026-06-24 10:28
Status: #baby 
Tags: [[graphics theory]] [[light theory]]
# Adding ambient occlusion

If you have an occlusion that's part of the your model. In this example we have a texture of an astroid, were the green component is roughness, blue is the metalness and the red is the occlusion. 

![[astroid-picture.png]]

You an take an straight colour from your albedo and multiple it by your occlusion factor. Allow you to have varing about of ambient light. 

```c
vec3 ambient = occlusion * (baseColour.rgb * 0.01);
```

Note, not all models have this so by default your occlusion factor should be 1 for every pixel.
# References
##### Main Notes
[[What is ambient light]]
#### Source Notes
