2026-06-14 14:12
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# Biased estimators

Not all estimators of a integal have the same expected value of the integal. These are considered **baised** estimators, where the difference is worked by a simple subtraction
$$\beta = E[F] - \int f(x)dx$$
It maybe better to use an estimator that's baised if it can get close to the expected value much faster than an non-basied estimator. 

So consider working out the mean value of a uniform distribution over 0 to 1 with random sampling of $X_i$ You could use two estimators 1 baised and 1 not-baised.
$$\frac{1}{n}\sum_{i = 1}^{n}X_i$$
or you could use this baised estimator 
$$\frac{1}{2}\max(X_1,X_2,...X_n)$$
The non-baised one variance with an order of $O(n^{-1})$ the second estimator expect value is
$$0.5\frac{n}{n+1}\neq{0.5}$$
but has a variance of $O(n^{-2})$ which is much better as you can see in this graph, the red is the first estimators variance and the blue is the second estimators variance.

![[converging-variance-2.png]]

So some estimators have nice properties a the samples $n$ for to infinity the errors found in the bias go to zero. This what we call a **consistent** estimator. Example of one is the biased estimator in this note.
# References
##### Main Notes
[[The Monte Carlo Estimator]]
[[Working out variance and the MSE for estimators]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]
