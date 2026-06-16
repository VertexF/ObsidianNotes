2026-06-16 12:40
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# Multiple Importance sampling composition

**Multiple importance sampling (MIS)** is done with PDF that are all valid for importance sampling the integrand with nonzero probability for generating a sample anywhere that the integrand is nonzero. However, not every PDF over has to be be nonzero if the functions nonzero. Only one must be.

Now we have **MIS compensation** which can further reduce variance. It's motivated by the fact that if all the sampling distributions allocates some probability to sampling regions, where the intergrand's value is small, this leads the intergrand to be undersampled were it's high and over sampled were it's low.

So since you can with MIS have one or more PDF be zero but not all. We can sharpen one or more (but not all) PDFs to have zero probability in areas they had probability.  A new sampling PDF $p'$ can be defined as
$$p'(x) = \frac{\max(0, p(x) - \delta)}{\int \max(0, p(x) - \delta)dx}$$
where $\delta$ is a fixed value.

This is easy to add for a tabularized sampling distribution. This can be used to help with sampling environment map light sources.
# References
##### Main Notes
[[Multiple Importance sampling composition]]
[[What is importance sampling]]
[[The problem with Multiple Importance Sampling]]
[[How to solve the multiple Importance sampling issue]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]