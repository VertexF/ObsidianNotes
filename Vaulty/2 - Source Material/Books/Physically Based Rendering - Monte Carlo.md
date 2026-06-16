# Reference https://www.pbr-book.org/4ed/Monte_Carlo_Integration

##### Introduction
When solving rendering equation like the **light transport equation** we need a method to go about solving these intergals that can be higher dimensional and discontinuous. Other method to solve intergals break down in this case but the **Monte Carlo** approach scale nicely with any type of intergal that we need to solve in rendering.

What this means is that we just need to solve $f(x)$ for a given intergrand with Monte Carlo with a random value. This gives us an estimate value on a given integral.

When using randomness in algorithms you have two types of categories. **Las Vegas** and **Monte Carlo**. 

**Las Vegas** algorithms are those that use randomness but always give the same return. For example the entry pivot point in Quicksort, the result here being that the array is sorted no matter the value picked. 

**Monte Carlo** happens when you use randomness and you get different values each time, which eventually converge on the correct value on average when you re-run a Monte Carlo more than once on the same input.
##### Background and probability review
In probability it's common practice to use $X$ for random variable with the expection of a few greek letters that mark out special random numbers. Random numbers are always picked from a **domain** and these **domains** can either be **discrete** (e.g finite set of possibilities) or **continuous** (e.g all real number $R$) When you use a function $f$ with a random number $X$ this will result a new random number $Y$ with $f(X) = Y$

When rolling a dice, you are selecting from a set of numbers randomly $X_i \in \{1, 2, 3, 4, 5, 6\}$ each has the probability of $p_i = \frac{1}{6}$ and the sum is $\sum_i$ is always 1. If a random number for probability has an equal chance we say that's **uniform**. If we have a function $p(X)$ that gives discrete random variable's probability this is called the **probability mass function (PMF)**, and so we could write this as $p(X) = \frac{1}{6}$ in this case.

You can have **joint probability** of $p(X,Y)$ of two random variables which don't depend each other, you can work out the probability as a product of 2 functions.
$$p(X,Y) = p(X)p(Y)$$

If you have a **dependent** random variable. For example say, if we are randomly picking out a marbles from a bag and these marbles are coloured differently. Picking one out affects the probability for picking out another. In this case of joint probability we have two marbes of $X$ and $Y$. The conditional probability of $Y$ is given a value of $X$ is written as

$$p(X, Y) = p(X)p(X|Y)$$
There might be situation is were a variable $X$ is conditional on many random values. 

You have a important random variable called the **canonical uniform random variable** written as $\xi$ This value takes on all vaues in it's domain $[0, 1)$ independently and with uniform probability. This $\xi$ is just an RNG in software. We need this for 2 reason.
1) First it's trivial get these values in software.
2) We also need to take random values from $\xi$ that gets value $X_i$ giving us the maths notation.
$$\sum_{j=1}^{i-1}p_j \leq \xi \lt \sum_{j=1}^{i}p_j$$

So we might for example want to sample the illumination from each light in the scene on it's power $\Phi_i$ is relative to the entire scene
$$p_i = \frac{\Phi_i}{\sum_j\Phi_j}$$
The sum of the lights will add up to 1.

The **cumulative distribution function(CDF)** $P(x)$ of a random variable is the probability that a value from the variable's distribution is less than or equal to some value of $x$ 
$$P(x) = Pr\{X \leq x\}$$
In the dice example this would be $P(2) = \frac{1}{3}$ because out of the six possibilities 1 and 2 are less than or equal to 2, meaning that we have a $\frac{1}{3}$ chance of that happening.

**Continuous random variables** can be also be distributed over non-uniform domains. 

When you have a set like $X_i \in \{1, 2, 3, 4, 5, 6\}$ the constant value is equal to the **probability mass function (PMF)** $\frac{1}{6}$ meaning that when we take the integral over probability of us taking a random number from the set $X_i \in \{1, 2, 3, 4, 5, 6\}$ we are asking what is the probability of selection of any these number added all together. This is also true for all finite sets.

For infinite sets you use the continuous extension of $\sum$ aka $\int$ 

The **probability density function (PDF)** describes the relative probability of a random variable taking a particular value and is continous analog of the **probability mass function (PMF)** The **PDF** $p(x)$ is the derivative of the random variable's **cumulative distribution function(CDF)**.
$$p(x) = \frac{dP(x)}{dx}$$
So the **cumulative distribution function(CDF)** is defined by the intergal over the **probability density function (PDF)**. If we intergate over a sub section of the **probability density function (PDF)** we will get a value less than 1.

Think about it like this with the set $X_i \in \{1, 2, 3, 4, 5, 6\}$ in this uniform distribution of selecting number out of this set. If we only intergate over and up to the values of 2. Then the **cumulative distribution function(CDF)** will be $\frac{1}{3}$ if you integate over the whole of the uniform distribution the (**CDF**) will be 1.

So lets write that down mathemically, so the chances of a values falling within a range of a given value over a **probability density function (PDF)** is written as
$$Pr\{x \in[a, b]\} = \int_{a}^{b}p(x)dx = P(b) - P(a)$$This follows the funimental theorem of calculus.

Finally the **proabability mass function (PMF)** and **probability density function** are the same. The latter is for discrete values like, 1, 2, 3 and the other is for continuous values like all the real values over the domain of $[0, 1]$.
##### Expected values
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
##### The Monte Carlo Estimator
We can now define the Monte Carlo estimator as something that approximates the value of an arbitrary integral. Over a domain $[a, b]$ were $X_i \in [a, b]$ 
$$F_n = \frac{b - a}{n}\sum_{i = 1}^{n}f(X_i)$$
The $F_n$ is equal to the integral. This is also for a 1D intergal. 

If we consider the whole range of the domain we can do some alegbraic manipulation. We also need to make sure that we understand that **probability density function (PDF)** corresponding to the random variable $X_i$ must be equal to $1/(b-a)$ since $p$ must not only be a constant but also integrate to 1 over the whole domain $[a, b]$

We are asking here what is the expect value of the result of the intergal.
$$E[F_n] = E[\frac{b - a}{n}\sum_{i = 1}^{n}f(X_i)]$$
$$= \frac{b - a}{n}\sum_{i=1}^{n}E[f(X_i)]$$
Here we are taking the expected value and replacing it with what it would be if we just integrated over that range instead.
$$= \frac{b - a}{n}\sum_{i=1}^{n}\int_{a}^{b}f(x)p(x)dx$$
Here we are using the **probability density function (PDF)** we talked about before, to simplify. This intergates to 1 remember over the whole domain therefor get 1/n
$$= \frac{1}{n}\sum_{i=1}^{n}\int_{a}^{b}f(x)dx$$
$$= \int_{a}^{b}f(x)dx$$
This works for any intergal of higher dimensional. The only difference here is that the $\frac{b - a}{n}$ goes over all the ranges in all the dimensions.
##### Reducing errors in the Monte Carlo estimator
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
##### Error in Monte Carlo estimators
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
##### Comparing estimators
The linear decrease in variance with the increasing number of samples makes it easy to compare estimators. How do we compare estimators then?

Well consider this example were we have 2 estimators, A and B. 
1) The A estimator has half the variance of B
2) B but is 3 times faster than A
B is preferred over A because we can 3x the number of samples, which would reduce the variance by 3x, beating A.

To work out the effciency we us this equation for a estimator $F$
$$\epsilon[F] = \frac{1}{V[F]T[F]}$$
Were $V[F]$ is the varience and $T[F]$ is the time to compute.
##### Baised estimators
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
##### Working out variance and the MSE for estimators
Remember that an estimator is trying to work out the value of an intergal. The **mean squared error (MSE)** is defined as the expectation of the squared difference of an estimator and the true value.
$$MSE[F] = E[(F-\int f(x)dx)]$$
You really can't work out the variance or MSE **mean squared error** with rendering equation estimators. To work out both you need to sample some random variables $X_i$ using the equation
$$V[F] = E[(F - E[F])^2]$$
We just take a set of independent random variable $X_i$ and sample over the equation above were $F$ take in the samples. You can't figure out the variance analytically because that would require us to analytically solve the intergal the estimator is estimating to know it's variance from the actual value.

So if the **sample mean** if we take the average samples with $Avg = (1/n)\sum X_i$ then the **sample variance** is
$$\frac{1}{n-1}\sum_{i = 1}^{n}(X_i - Avg)^2$$
So the division by $n - 1$ rather than $n$ is the **Bessel's correction** and ensures that variance is unbiased estimate of the variance.

The sample variance is an estimation on the actual variance. So it's sometimes useful to figure out the variance of the sample variance. Consider we have a random variable that has a value of 1 $99.99\%$ of the time and 1,000,000 $0.01\%$ of the time. If we took 10 random samples it's extremely like that it would tell use that we have zero variance on our sample variance, which is wrong.

So if we can accuratly estimate the integral $est \approx \int f(x)dx$ for example using loads of samples. We can estimate the **mean square error (MSE)** like this.
$$MSE[F]\approx \frac{1}{n}\sum_{i=1}^{n}(f(X_i) - est)$$
You can use this to diff images for a referrence image to see how accurate you are.
##### Introduction on improve efficiency for Monte Carlo
Yes you can use straight forward Monte Carlo estimator, and with enough computation time you should get a clean image generated. However, ain't nobody got time for that. So when we talk about efficiency in Monte Carlo we are trying to make more use of the samples we have taken.
##### Stratified sampling
One idea to reduce variance to worry about the placement of the samples to better capture the integrand, aka not missing important features. When we say stratified we are really talking about the arrangement of these sample. Stratifing sampling decomposes the integration domain into regions and pslaces samples in each one.

Stratified sampling subdivides the integration domain $\Lambda$ into n nonoverlapping regions called stratums and they complete the orignal domain
$$\bigcup_{i=1}^{n}\Lambda_i = \Lambda$$
To draw samples from $\Lambda$ in $n_i$ stratums with pixel density of $p_i$ for each stratrum. Each area is divided into $k\times k$ grid and a sample is drawn uniformally within each grid. This avoids clumping samples together. 

With [[Reducing errors in the Monte Carlo estimator|Monte Carlo estimator]] we can do this over a stratum $\Lambda_i$ 
$$F_i = \frac{1}{n_i}\sum_{j= 1}^{n_i}\frac{f(X_{ij})}{p(X_{ij})}$$
Were $X_{ij}$ within a nested for loop for each stratum by pixel density $p_i$ So if you add all the strata together you get this where $F=\sum_i v_i F_i$ where $v_i$ is the ith fractional volume.
##### Variance in stratum sampling 
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

##### How to calculate the mean of a continous function
To continue you need to understand thing you need to know what the **mean** of a function is over a continuous domain. 
$$Q = \frac{1}{b-a} \int_{a}^{b}f(x)dx$$
This states that over a continous function of $f(x)$ we take all the valus of $y$ that comes from $f(x)$ and add them all together. Then devide by the fraction. This is almost exactly the as the discrete version of meaning of **mean** we are just doing over a continous function, that's why we need to intergal over a domain of $[a, b]$ 
$$V[F] = \frac{1}{n}[\sum v_io_i^2 + \sum v_i(\mu_i - Q)^2]$$
 $Q$ is the mean of the $f$ over the whole domain. We are doing this compare the stratums expect value to the whole expected value over the entire domain.

There are two things to notice about the equation.
1) First because of the square, the right hand part can't be negative. 
2) Second it should that stratified sampling can never increase the variance. 
Stratification always reduces variance until the right-hand sum is exactly 0. When we have the same **mean** across the entire stratum, then the right-hand sum is 0. This means we need to increase the right hand-sum to improve the stratum based sampling. So the more different the **mean** of each stratum the better. So **compact** strata are desirable if one does not know anything about the function $f$. If the strata are wide, they will contain more variation (which is good) and will have $\mu_i$ close to the true **mean** of $Q$

![[noisy-image-2.png]]
![[cleaner-stratified-image-2.png]]

We basically get these gains at almost no cost of computation. 
##### The problem with stratified sampling
The main downside of stratified sampling is that it suffers in the same way as standard numerical quadrature, it degrades with increased dimension. 

For instance, if we had $D$ dimensions with $S$ strata per dimension that would require $S^D$ samples which becomes too expensive. 

Fortunately, it's possible stratify some of the dimensions independenly and then randonly associate samples from different dimensions. When you do this you need to make sure that it is done in away that stratifies dimensions that tend to be most highly correlated in their effect on the value of the integrand.
##### What is importance sampling
Importances sampling takes advantage of the Monte Carlo estimator. [[Reducing errors in the Monte Carlo estimator]] 
$$F_n = \frac{1}{n}\sum_{i = 1}^{n}\frac{f(X_i)}{p(X_i)}$$
When you take the **probability density function (PDF)** and sample over it were the magnitude of the integrand is relatively large. This is one of the most commonly used and helpful way to reduce variance in rendering since it's easy apply and very effective.

To see why the samplying distributions reduces variances let first look at a function that has a properation relation with a PDF $p(x) \propto f(x)$ or if $f(x)$ times a constant is equal to the PDF $p(x) = cf(x)$ To normalize the PDF it requires us to
$$c = \frac{1}{\int f(x)dx}$$
A normalized function is when the intergal is equal to 1 over the entire domain. Of course in our case we would need to know the integral of the value integral which what we estimating. If we could sample from this distribution wach of the sum, in the estimator we would have the value of
$$\frac{f(X_i)}{p(X_i)} = \frac{1}{c} = \int f(x)dx$$
The variance here is zero which is wrong. Since if we could just integrate the integal directly. However, if the density $p(x)$ can be found that is similar to the shaope to $f(x)$ the variance is reduced. 

If we consider a function Gaussian function $f(x) = e^{-1000(x-1/2)^2}$ over the domain of $[0, 1]$ which would look like this

![[guassian-plotted-function-2.png]]

If we take the intergal over the function  $f(x) = e^{-1000(x-1/2)^2}$ and sample randomly with a standard Monte Carlo estimator the variance is about 0.0365 since it's very likely we select a value that is almost 0.

Instead we could draw from a piecewise-constant distibution
$$p(x) =  \lbrace{0.1 x \in [0, 0.45]}$$
$$p(x) =  \lbrace{9.1 x \in [0.45, 0.55]}$$
$$p(x) =  \lbrace{0.1 x \in [0.55, 1)}$$

This is plotted below.
![[piecewise-gauss-2.png]]

So if this estimator is used instead of a regular Monte Carlo, we get about a 6.7x variance reduction, when selection 6 from this distrubtion.
$$F_n = \frac{1}{n}\sum_{i = 1}^{n}\frac{f(X_i)}{p(X_i)}$$
![[selection-of-points-2.png]]
##### The problem with Importance Sampling
The problem is pretty straight forward if you have the Monte Carlo estimator with a probability distribution function.
$$F_n = \frac{1}{n}\sum_{i = 1}^{n}\frac{f(X_i)}{p(X_i)}$$
It's possible that your PDF **probability density function** with a bad distribution and that the variance actually increases. 
##### The problem with Multiple Importance Sampling
When we are rendering we often are faced with integral result are a products of other integrals. So when you want to [[What is importance sampling|importance sampling]] you need to consider the products of the intergals. This is specially important with the [[What is the light transport equation|light transport equation]]. 

So lets assume we have two good PDFs ($p_a$ and $p_b$) for the two functions ($f_a$ and $f_b$). In practice this wont normally be the case. Lets assume we use $p_a$ as our PDF. We would have something like this.
$$\frac{f(X)}{p_a(X)} = \frac{f_a(X)f_b(X)}{p_a(X)} = cf_b(X)$$
where $c$ is a constant equal to the interfal of $f_a$ since $p_a(x)$ and $f_a(x)$ are proportional $p_a(x) \propto f_a(x)$. In this case, the variance would proportional to the variance of $f_b$ which could be high. Usually, in real rendering situations when you selection 1 PDF like the variance is very high.

We can't just add two estimators to together, with two different PDF because variance is additive means that you can't reduce the variance of a bad estimator by adding a good one to it.

One shortcoming of **multiple Importance sampling (MIS)** 
##### How to solve the multiple Importance sampling issue
There is a a way with **multiple Importance sampling (MIS)** to solve this. This adds weight to our different PDFs for each intergand and by then we can select good samples that match the overall shape of the intergal. Specialised sampling routines that only account for unusual special cases are even encoraged, as they reduce variance when those cases occur with relatively little cost in general.

So with two sampling distributions $p_a$ and $p_b$ and single sample taken from each one were $X$ over $p_a$ and $Y$ over $p_b$ you have this **Multiple Importance Sampling (MIS)** is
$$F_n = \sum_{i = 1}^{n}(\frac{1}{n_i}\sum_{j=1}^{n_i}w_i(X_{ij})\frac{f(X_{ij})}{p_i(X_{ij})})$$
The full set of conditions and weighting functions for the estimator to be unbiased are that they sum 1 $f(x) \neq 0, \sum_{i = 1}^{n} w_i(x) = 1$ and that $w_i(x) = 0$ if $p_i(x) = 0$

So making the weighting function be $w_i(X) = 1/n$ you just get the same additive estimator issues you had earlier with [[The problem with Multiple Importance Sampling]] so you need to weighting function to be high when a sample reduces variance and low when it increases variance.
##### The balance heuristic function
So we need to fo something called a **balance heuristic function** which takes into account all the ways samples have been generated, rather than just the particular one that was used. This is what **balance heuristic function** looks like for the $i$th sampling technique:
$$w_i(x) = \frac{n_ip_i(x)}{\sum_{j}n_ip_i(x)}$$
So when you have two estimators $p_a$ and $p_b$ and you sample with this technique you get the estimator.
$$\frac{f(X)}{p_a(X) + p_b(X)} + \frac{f(Y)}{p_a(Y) + p_b(Y)}$$
Where $X$ and $Y$ are random samples, were $f$ is divided by the sum of the **PDFs** rather than just one that generated teh sample. Thus, if $p_a$ generates a sample with a low probability at a point where the $p_b$ has a higher probabilitym then dividing by $p_a(X) + p_b(X)$ reduces the sample's contribution. Meaning that in this situation $p_a$ is downwieght. As long as one of these functions has a probability of sampling a point that is large in the intergal the the **Multi Important Sampling (MIS)** weights can lead to a significan reduction in variance.

So for most part you don't need more than 2 PDF when doing **Multi Important Sampling (MIS)** if one **PDF** that matches the integrand very well then, **MIS** can reduce the over variance. However **MIS** is better almost all the time so it's good to us. 

The code would look like this
```c++
float balanceHeuristics(int sampleX, float probDistFuncA int sampleY float probDistFuncB)
{
	return (sampleX * probDistFuncA) / (sampleX * probDistFuncA + sampleY * probDistFuncB);
}
```
##### The power heuristics
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
##### The single sample model
You can single sample for a **Multi Important Sampling (MIS)** approach. So for  a sampling technique of $p_i$ is chosen from a set of techniques with the probability of $q_i$ and a sample $X$ is drawn from $p_i$ then the **single sample estimator** is
$$\frac{w_i(X)}{q_i} \frac{f(X)}{p_i(X)}$$
This given an unbiased estimate of the intergal. For a single sample model, the [[The balance heuristic function]] for the weighting function.
##### MIS Compensation
**Multiple importance sampling (MIS)** is done with PDF that are all valid for importance sampling the integrand with nonzero probability for generating a sample anywhere that the integrand is nonzero. However, not every PDF over has to be be nonzero if the functions nonzero. Only one must be.

Now we have **MIS compensation** which can further reduce variance. It's motivated by the fact that if all the sampling distributions allocates some probability to sampling regions, where the intergrand's value is small, this leads the intergrand to be undersampled were it's high and over sampled were it's low.

So since you can with MIS have one or more PDF be zero but not all. We can sharpen one or more (but not all) PDFs to have zero probability in areas they had probability.  A new sampling PDF $p'$ can be defined as
$$p'(x) = \frac{\max(0, p(x) - \delta)}{\int \max(0, p(x) - \delta)dx}$$
where $\delta$ is a fixed value.

This is easy to add for a tabularized sampling distribution. This can be used to help with sampling environment map light sources.