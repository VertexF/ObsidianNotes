2026-06-16 12:36
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# The power heuristics funciton

The power heuristics reduces varianace even further. For an exponent $\beta$, the power heuristic is
$$w_i(x) = \frac{(n_ip_i(x))^{\beta}}{\sum_{j} n_jp_j(x)}$$
This is very similar to [[The balance heuristic function]] but it futher reduces contributions of relatively low probabilities. The implementation of $\beta = 2$ is completely fine.

```c++
float powerHeuristic(int sampleX, float probDistFuncA, int sampleY, float probDistFuncB)
{
	float f = sampleX * probDistFuncA;
	float g = sampleY * probDistFuncB;
	return f * f / (f * f) + (g * g);
}
```
# References
##### Main Notes
[[The balance heuristic function]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]