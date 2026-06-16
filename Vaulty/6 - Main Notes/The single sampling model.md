2026-06-16 12:37
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# The single sampling model

You can single sample for a **Multi Important Sampling (MIS)** approach. So for  a sampling technique of $p_i$ is chosen from a set of techniques with the probability of $q_i$ and a sample $X$ is drawn from $p_i$ then the **single sample estimator** is
$$\frac{w_i(X)}{q_i} \frac{f(X)}{p_i(X)}$$
This given an unbiased estimate of the intergal. For a single sample model, the [[The balance heuristic function]] for the weighting function.
# References
##### Main Notes
[[Multiple Importance sampling composition]]
[[What is importance sampling]]
[[The problem with Multiple Importance Sampling]]
[[How to solve the multiple Importance sampling issue]]
[[The balance heuristic function]]
[[The power heuristics funciton]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]