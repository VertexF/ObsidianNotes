2026-06-16 12:19
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# Variance in stratified sampling

So the true value of the integrand in stratum $i$ 
$$\mu_i = E[f(X_{ij})] = \frac{1}{v_i}\int_{\Lambda_i}f(x)dx$$
and the variance in this stratum
$$\sigma_i^2=\frac{1}{v_i}\int_{\Lambda_i} (f(x) - \mu_i)^2dx$$
This mean that if you take $\sigma_i^2$ and divide it by $\mu_i$ you get the variance per stratum. To get the overall variance you add them all together.
$$V[F]=V[\sum v_iF_i]$$
$$= \sum V[v_iF_i]$$
$$=\sum v_{i}^2V[F_i]$$
$$= \sum \frac{v_{i}^2\sigma_{i}^2}{n_i}$$
If we make the reaonable assumption that the number of samples $n_i$ is proportional. Meaning the relationship between the samples and volume of stratum are the same. Then we can say that $n_i = v_i n$ and the variance of the over estimator is
$$V[F_n] = \frac{1}{n}\sum v_i\sigma_{i}^2$$
To compare this result to variance with a stratum. We need to choose a none stratified sample is the equivalent to choosing a ransom stratum $I$ then choosing a random sample $X$ in $\Lambda_I$. This would mean that $X$ is conditional on $I$. This become a condition proproability [[Background and probability review]] explain the basics.
$$V[F] = \frac{1}{n}[\sum v_io_i^2 + \sum v_i(\mu_i - Q)^2]$$
 $Q$ is the mean of the $f$ over the whole domain. We are doing this compare the stratums expect value to the whole expected value over the entire domain.

There are two things to notice about the equation.
1) First because of the square, the right hand part can't be negative. 
2) Second it should that stratified sampling can never increase the variance. 
Stratification always reduces variance until the right-hand sum is exactly 0. When we have the same [[How to calculate the mean of a continous function|mean]] across the entire stratum, then the right-hand sum is 0. This means we need to increase the right hand-sum to improve the stratum based sampling. So the more different the **mean** of each stratum the better. So **compact** strata are desirable if one does not know anything about the function $f$. If the strata are wide, they will contain more variation (which is good) and will have $\mu_i$ close to the true **mean** of $Q$

![[noisy-image-2.png]]
![[cleaner-stratified-image-2.png]]

We basically get these gains at almost no cost of computation. 
# References
##### Main Notes
[[How to calculate the mean of a continous function]]
[[What is stratified sampling]]
[[The problem with stratified sampling]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]