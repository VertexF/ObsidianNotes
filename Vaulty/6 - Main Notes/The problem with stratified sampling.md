2026-06-16 12:25
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# The problem with stratified sampling

The main downside of stratified sampling is that it suffers in the same way as standard numerical quadrature, it degrades with increased dimension. 

For instance, if we had $D$ dimensions with $S$ strata per dimension that would require $S^D$ samples which becomes too expensive. 

Fortunately, it's possible stratify some of the dimensions independenly and then randonly associate samples from different dimensions. When you do this you need to make sure that it is done in away that stratifies dimensions that tend to be most highly correlated in their effect on the value of the integrand.
# References
##### Main Notes
[[What is stratified sampling]]
[[Variance in stratified sampling]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]