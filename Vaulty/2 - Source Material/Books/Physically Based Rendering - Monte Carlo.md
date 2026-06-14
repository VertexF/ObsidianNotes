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
To draw samples from $\Lambda$ in $n_i$ stratums with pixel density of $p_i$