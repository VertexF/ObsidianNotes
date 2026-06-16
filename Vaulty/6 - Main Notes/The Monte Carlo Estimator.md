2026-06-14 14:02
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# The Monte Carlo Estimator

We can now define the Monte Carlo estimator as something that approximates the value of an arbitrary integral. Over a domain $[a, b]$ were $X_i \in [a, b]$ 
$$F_n = \frac{b - a}{n}\sum_{i = 1}^{n}f(X_i)$$
The $F_n$ is equal to the integral. This is also for a 1D intergal. 

If we consider the whole range of the domain we can do some alegbraic manipulation. We also need to make sure that we understand that **probability density function (PDF)** corresponding to the random variable $X_i$ must be equal to $1/(b-a)$ since $p$ must not only be a constant but also integrate to 1 over the whole domain $[a, b]$

We are asking here what is the expect value of the result of the intergal.
$$E[F_n] = E[\frac{b - a}{n}\sum_{i = 1}^{n}f(X_i)]$$
$$= \frac{b - a}{n}\sum_{i=1}^{n}E[f(X_i)]$$
Here we are taking the expected value and replacing it with what it would be if we just integrated over that range instead.
$$= \frac{b - a}{n}\sum_{i=1}^{n}\int_{a}^{b}f(x)p(x)dx$$
Here we are using the **probability density function (PDF)** we talked about before, to simplify to 1 because we are using the entire range of the domain. This intergates to 1 remember over the whole domain therefor get 1/n too.
$$= \frac{1}{n}\sum_{i=1}^{n}\int_{a}^{b}f(x)dx$$
$$= \int_{a}^{b}f(x)dx$$
This works for any intergal of higher dimensional. The only difference here is that the $\frac{b - a}{n}$ goes over all the ranges in all the dimensions.
# References
##### Main Notes
[[Error in Monte Carlo estimators]]
[[Background and probability review]]
[[Expected values]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]