2026-06-14 14:10
Status: #baby 
Tags: [[graphics theory]] [[ray tracing]]
# Working out variance and the MSE for estimators

Remember that an estimator is trying to work out the value of an intergal. The **mean squared error (MSE)** is defined as the expectation of the squared difference of an estimator and the true value.
$$MSE[F] = E[(F-\int f(x)dx)]$$
You really can't work out the variance or MSE **mean squared error** with rendering equation estimators. To work out both you need to sample some random variables $X_i$ using the equation
$$V[F] = E[(F - E[F])^2]$$
We just take a set of independent random variable $X_i$ and sample over the equation above were $F$ take in the samples. You can't figure out the variance analytically because that would require us to analytically solve the intergal the estimator is estimating to know it's variance from the actual value.

So if the **sample mean** if we take the average samples with $Avg = (1/n)\sum X_i$ then the **sample variance** is
$$\frac{1}{n-1}\sum_{i = 1}^{n}(X_i - Avg)^2$$
So the division by $n - 1$ rather than $n$ is the **Bessel's correction** and ensures that variance is unbiased estimate of the variance.

The sample variance is an estimation on the actual variance. So it's sometimes useful to figure out the variance of the sample variance. Consider we have a random variable that has a value of 1 $99.99\%$ of the time and 1,000,000 $0.01\%$ of the time. If we took 10 random samples it's extremely like that it would tell use that we have zero variance on our sample variance, which is wrong.

So if we can accuratly estimate the integral $Est \approx \int f(x)dx$ for example using loads of samples. We can estimate the **mean square error (MSE)** like this.
$$MSE[F]\approx \frac{1}{n}\sum_{i=1}^{n}(f(X_i) - Est)$$
You can use this to diff images for a referrence image to see how accurate you are.
# References
##### Main Notes
[[Biased estimators]]
[[Comparing estimators]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]