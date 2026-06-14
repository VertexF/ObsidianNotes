2026-06-14 14:01
Status: #baby 
Tags: [[graphics theory]] [[maths]]
# Expected values

The **expected value** of $E_p[f(x)]$ of a function $f$ is defined by the average value of a function over a distribution of values $p(x)$ over its domain $D$
$$E_p[f(x)] = \int_Df(x)p(x)dx$$
So if we are trying to find the expected value of the cosine function between 0 and $\pi$ where $p$ is uniform. Because the **probability density function (PDF)** must integrate to 1 over the domain $p(x) = \frac{1}{\pi}$ 
$$E[\cos(x)] = \int_{0}^{\pi}\frac{\cos(x)}{\pi} = \frac{1}{\pi}(\sin(\pi) - \sin(0)) = 0$$
![[cosine-over1-pi-1.png]]

This is because if we add up the average of all these values which go from 1 to -1 the average will be 0.

This is true because it's a derivitive property.
$$E[af(x)] = aE[f(x)]$$

As the expected value is a scalar then this hopefully should help you understand what's happening below.

If you take the sum of all the expected values of function that passes in random variables, that's the same as adding all the values of that function and getting the expected value from that. 
$$E[\sum_{i=1}^{n}f(X_i)] = \sum_{i=1}^{n}E[f(X_i)]$$
# References
##### Main Notes
[[Background and probability review]]
[[The Monte Carlo Estimator]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]