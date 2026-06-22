2026-06-16 12:35
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# The balance heuristic function

So we need to fo something called a **balance heuristic function** which takes into account all the ways samples have been generated, rather than just the particular one that was used. This is what **balance heuristic function** looks like for the $i$th sampling technique:
$$w_i(x) = \frac{n_ip_i(x)}{\sum_{j}n_ip_i(x)}$$
So when you have two estimators $p_a$ and $p_b$ and you sample with this technique you get the estimator.
$$\frac{f(X)}{p_a(X) + p_b(X)} + \frac{f(Y)}{p_a(Y) + p_b(Y)}$$
Where $X$ and $Y$ are random samples, were $f$ is divided by the sum of the **PDFs** rather than just one that generated the sample. Thus, if $p_a$ generates a sample with a low probability at a point where the $p_b$ has a higher probability then dividing by $p_a(X) + p_b(X)$ reduces the sample's contribution. Meaning that in this situation $p_a$ is downwieght. As long as one of these functions has a probability of sampling a point that is large in the intergal the the **Multi Important Sampling (MIS)** weights can lead to a significan reduction in variance.

So for most part you don't need more than 2 PDF when doing **Multi Important Sampling (MIS)** if one **PDF** that matches the integrand very well then, **MIS** can reduce the over variance. However **MIS** is better almost all the time so it's good to us. 

The code would look like this
```c++
float balanceHeuristics(int sampleX, float probDistFuncA int sampleY float probDistFuncB)
{
	return (sampleX * probDistFuncA) / (sampleX * probDistFuncA + sampleY * probDistFuncB);
}
```
# References
##### Main Notes
[[How to solve the Multiple Importance Sampling issue]]
[[The power heuristics funciton]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]
