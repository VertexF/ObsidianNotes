2026-06-22 14:47
Status: #baby 
Tags: [[graphics theory]] [[light theory]]
# Attenuation function

We can't use the standard mathematical equation, this is because it's impractical implement.
1) The division by the square distance can lead to divides by 0, when an object touches the light source.
2) The influence sphere of each light is infinite. ($\frac{I}{d^2}$ is asymptotic, it never reaches 0) which mean that to correctly shade a pixel we need to evaluate every light in the world.
The first issue that we are have can be solved by making the assumption that we are have an area light with a small area of 1cm.
$$E = \frac{I}{max(d^2, {0.01}^2)}$$
The second issue we need to introduce a sphere influence each light. This allows us to use the debug sphere to show the influence. We can also cull lights more aggrestively using this extra piece of information.  With a influence sphere you can manually tweak the radius of the light which might help out artists.

Mathematically, the illuminance of a light should smoothly reach zero at the limit definied by a influence radius. In this [paper](Brian Karis, 2013. Real Shading in Unreal Engine 4. [https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf](https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf)) we take the inverse square function and we use a function that allow the ligths influence to remain unaffected. We have this in with this equation were the radius $r$
$$E = \frac{I}{max(d^2, {0.01}^2)} \left< 1 - \frac{d^4}{r^4} \right>^2$$
Note this equation is purely for the sphere of influence and it's nothing to do with a spot light itself. 
# References
##### Main Notes
[[The spot light]]
[[The point light]]
[[Punctual lights theory]]
[[Lighting unit basics]]
#### Source Notes
[[Filament PBR - Lighting]]