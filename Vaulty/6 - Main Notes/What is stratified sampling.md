2026-06-16 12:17
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# What is stratified sampling

One idea to reduce variance to worry about the placement of the samples to better capture the integrand, aka not missing important features. When we say stratified we are really talking about the arrangement of these sample. Stratifing sampling decomposes the integration domain into regions and pslaces samples in each one.

Stratified sampling subdivides the integration domain $\Lambda$ into n nonoverlapping regions called stratums and they complete the orignal domain
$$\bigcup_{i=1}^{n}\Lambda_i = \Lambda$$
To draw samples from $\Lambda$ in $n_i$ stratums with pixel density of $p_i$ for each stratrum. Each area is divided into $k\times k$ grid and a sample is drawn uniformally within each grid. This avoids clumping samples together. 

With [[Reducing errors in the Monte Carlo estimator|Monte Carlo estimator]] we can do this over a stratum $\Lambda_i$ 
$$F_i = \frac{1}{n_i}\sum_{j= 1}^{n_i}\frac{f(X_{ij})}{p(X_{ij})}$$
Were $X_{ij}$ within a nested for loop for each stratum by pixel density $p_i$ So if you add all the strata together you get this where $F=\sum_i v_i F_i$ where $v_i$ is the ith fractional volume.
# References
##### Main Notes
[[The Monte Carlo Estimator]]
[[Variance in stratified sampling]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]
