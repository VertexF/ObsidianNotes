2026-06-14 14:05
Status: #baby 
Tags: [[graphics theory]] [[ray tracing]]
# Error in Monte Carlo estimators

Just because Monte Carlo converges at the right answer isn't enough we need to know how fast it finds the answer. The concept of **variance** is used to figure how far the estimator deviates from the expected value. 

If we have a Monte Carlo estimator $F$ the **variance** can from the **expected value** can be written like this
$$V[F] = E[(F - E[F])^2]$$
This below is here just here to show that if you work out all the squares then you'll get $a^2$
$$V[aF] = E[(aF - E[aF])^2] = a^2V[F]$$
Using the derivatives property for expected values in the [[Background and probability review]] we can change the first equation into something like this
$$V[F] = E[F^2] - E[F]^2$$
If the estimator is a sue of independent random variable like Monte Carlo $F_n$ This all works out to be the variance of the sum of the independent random variable variances
$$V[\sum_{i=1}^{n}X_i] = \sum_{i = 1}^{n}V[X_i]$$
The variance decreases linearly with the number of samples because we are summing them. The higher the variance power is, the lower the converagence. So with a variance number that's a square the converagence is $O(\frac{1}{\sqrt{n}})$  where $n$ is the number of samples.

You can see this if you watch a Monte Carlo estimator send out samples for each pixel. It rapidely improves the quality of the image at first but as the samples increase rapidly decreases, meaning that errors get harder to remove the fewer they are.

![[graphic-for-converagence-2.png]]

In this graph, as we increase in the x-axis (which would be number of samples) we are converaging (correcting mistakes), but at a increasingly slower rate.
# References
##### Main Notes
[[Expected values]]
[[The Monte Carlo Estimator]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]