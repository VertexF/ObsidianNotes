2026-06-16 12:28
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# What is importance sampling

Importances sampling takes advantage of the Monte Carlo estimator. [[Reducing errors in the Monte Carlo estimator]] 
$$F_n = \frac{1}{n}\sum_{i = 1}^{n}\frac{f(X_i)}{p(X_i)}$$
When you take the **probability density function (PDF)** and sample over it were the magnitude of the integrand is relatively large. This is one of the most commonly used and helpful way to reduce variance in rendering since it's easy apply and very effective.

To see why the samplying distributions reduces variances let first look at a function that has a properation relation with a PDF $p(x) \propto f(x)$ or if $f(x)$ times a constant is equal to the PDF $p(x) = cf(x)$ To normalize the PDF it requires us to
$$c = \frac{1}{\int f(x)dx}$$
A normalized function is when the intergal is equal to 1 over the entire domain. Of course in our case we would need to know the integral of the value integral which what we estimating. If we could sample from this distribution wach of the sum, in the estimator we would have the value of
$$\frac{f(X_i)}{p(X_i)} = \frac{1}{c} = \int f(x)dx$$
The variance here is zero which is wrong. Since if we could just integrate the integal directly. However, if the density $p(x)$ can be found that is similar to the shaope to $f(x)$ the variance is reduced. 

If we consider a function Gaussian function $f(x) = e^{-1000(x-1/2)^2}$ over the domain of $[0, 1]$ which would look like this

![[guassian-plotted-function-2.png]]

If we take the intergal over the function  $f(x) = e^{-1000(x-1/2)^2}$ and sample randomly with a standard Monte Carlo estimator the variance is about 0.0365 since it's very likely we select a value that is almost 0.

Instead we could draw from a piecewise-constant distibution
$$p(x) =  \lbrace{0.1 x \in [0, 0.45]}$$
$$p(x) =  \lbrace{9.1 x \in [0.45, 0.55]}$$
$$p(x) =  \lbrace{0.1 x \in [0.55, 1)}$$

This is plotted below.
![[piecewise-gauss-2.png]]

So if this estimator is used instead of a regular Monte Carlo, we get about a 6.7x variance reduction, when selection 6 from this distrubtion.
$$F_n = \frac{1}{n}\sum_{i = 1}^{n}\frac{f(X_i)}{p(X_i)}$$
![[selection-of-points-2.png]]
# References
##### Main Notes
[[The problem with Multiple Importance Sampling]]
[[How to solve the multiple Importance sampling issue]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]