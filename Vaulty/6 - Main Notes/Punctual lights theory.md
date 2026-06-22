2026-06-22 14:39
Status: #baby 
Tags: [[graphics theory]] [[light theory]]
# Punctual lights theory

Spot and punctual lights don't follow the inverse square and can be truly infinitestimally small.

If you care about performance it's good to use infinitesimally small punctual light when possible.

We can solve the first issue by making sure that the inverse square law has the term $E$ in the equation $L_{out} = f(v,l)E$ using that we express this idea with this equation.
$$E = L_{in} \left< n\cdot l \right> = \frac{I}{d^2} \left< n \cdot l \right>$$
The difference between a point and a spot light lies in how $E$ is computed, and in particular how luminous intensity $I$ is computed from the luminous power $\Phi$

Meaning when we are looking that the equation above if you're punctual light $\frac{I}{d^2}$ is the starting point before you begin writing anything. Note this value is **NOT** luminance ($L$) as that would per unit area, and not distance squared.
# References
##### Main Notes
[[Lighting unit basics]]
#### Source Notes
[[Filament PBR - Materials]]