2026-06-16 12:31
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# The problem with Multiple Importance Sampling

One shortcoming of **multiple Importance sampling (MIS)** when we are rendering we often are faced with integral result are a products of other integrals. So when you want to [[What is importance sampling|importance sampling]] you need to consider the products of the intergals. This is specially important with the [[What is the light transport equation|light transport equation]]. 

So lets assume we have two good PDFs ($p_a$ and $p_b$) for the two functions ($f_a$ and $f_b$). In practice this wont normally be the case. Lets assume we use $p_a$ as our PDF. We would have something like this.
$$\frac{f(X)}{p_a(X)} = \frac{f_a(X)f_b(X)}{p_a(X)} = cf_b(X)$$
where $c$ is a constant equal to the intergal of $f_a$ since $p_a(x)$ and $f_a(x)$ are proportional $p_a(x) \propto f_a(x)$. In this case, the variance would proportional to the variance of $f_b$ which could be high. Usually, in real rendering situations when you selection 1 PDF like the variance is very high.

We can't just add two estimators to together, with two different PDF because variance is additive means that you can't reduce the variance of a bad estimator by adding a good one to it.
# References
##### Main Notes
[[What is importance sampling]]
[[How to solve the Multiple Importance Sampling issue]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]