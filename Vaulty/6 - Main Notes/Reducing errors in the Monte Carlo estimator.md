2026-06-14 14:20
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# Reducing errors in the Monte Carlo estimator

We can remove some of the restrictions we have when we originally defined the [[The Monte Carlo Estimator]] This is important since carefully choosing the **probability density function (PDF)** were the samples are draw from, leads to a key technique for reducing errors in Monte Carlo. 

If the random variables $X_i$ are drawn from our **PDF** $p(x)$ then the estimator looks like this.
$$F_n = \frac{1}{n}\sum_{i = 1}^{n}\frac{f(X_i)}{p(X_i)}$$
So the only restriction is our **PDF** must not be nonzero for all values of $x$ where $|f(x)| > 0$ 

This is similarly to what we worked out before with our [[The Monte Carlo Estimator]] to see the expected value of the this estimator is for an integral
$$E[F_n] = E[\frac{1}{n}\sum_{i= 1}^{n}\frac{f(X_i)}{p(X_i)}]$$

Remember when we intergrate over **probability density function (PDF)** we get the **cumulative distribution function(CDF)**. 
$$= \frac{1}{n}\sum_{i= 1}^{n}\int_{a}^{b}\frac{f(x)}{p(x)}p(x)dx$$
$$= \frac{1}{n}\sum_{i= 1}^{n}\int_{a}^{b}f(x)dx$$
$$=\int_{a}^{b}f(x)dx$$
We can now understand why when we were [[Breaking down the light transport equation]] and we used a value of $1/(4\pi)$. We were samplying uniformly over a unit sphere which has the area of $4\pi$. Since **probability density function (PDF)** is normalized over the sampling domain it must have a constant value of $1/(4\pi)$. I hope you see the connection between the Monte Carlo estimator and the broken down light transport equation.

We can't use something like deterministic quadrature techniques because that increases exponentially in the dimensions with $n$ samples. Unlike with  in Monte Carlo.
# References
##### Main Notes
[[The Monte Carlo Estimator]]
[[Error in Monte Carlo estimators]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]