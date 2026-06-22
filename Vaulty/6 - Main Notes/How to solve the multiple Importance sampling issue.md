2026-06-16 12:33
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# How to solve the Multiple Importance Sampling issue

There is a a way with **multiple Importance sampling (MIS)** to solve this. This adds weight to our different PDFs for each intergand and by then we can select good samples that match the overall shape of the intergal. Specialised sampling routines that only account for unusual special cases are even encoraged, as they reduce variance when those cases occur with relatively little cost in general.

So with two sampling distributions $p_a$ and $p_b$ and single sample taken from each one were $X$ over $p_a$ and $Y$ over $p_b$ you have this **Multiple Importance Sampling (MIS)** is
$$F_n = \sum_{i = 1}^{n}(\frac{1}{n_i}\sum_{j=1}^{n_i}w_i(X_{ij})\frac{f(X_{ij})}{p_i(X_{ij})})$$
The full set of conditions and weighting functions for the estimator to be unbiased are that they sum 1 $f(x) \neq 0, \sum_{i = 1}^{n} w_i(x) = 1$ and that $w_i(x) = 0$ if $p_i(x) = 0$

So making the weighting function be $w_i(X) = 1/n$ you just get the same additive estimator issues you had earlier with [[The problem with Multiple Importance Sampling]] so you need to weighting function to be high when a sample reduces variance and low when it increases variance.
# References
##### Main Notes
[[The problem with Multiple Importance Sampling]]
[[What is importance sampling]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]
