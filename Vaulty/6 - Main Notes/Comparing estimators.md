2026-06-14 14:08
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# Comparing estimators

The linear decrease in variance with the increasing number of samples makes it easy to compare estimators. How do we compare estimators then?

Well consider this example were we have 2 estimators, A and B. 
1) The A estimator has half the variance of B
2) B but is 3 times faster than A
B is preferred over A because we can 3x the number of samples, which would reduce the variance by 3x, beating A.

To work out the effciency we us this equation for a estimator $F$
$$\epsilon[F] = \frac{1}{V[F]T[F]}$$
Were $V[F]$ is the varience and $T[F]$ is the time to compute.
# References
##### Main Notes
[[The Monte Carlo Estimator]]
[[Error in Monte Carlo estimators]]
[[Expected values]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]