2026-06-14 13:57
Status: #baby 
Tags: [[graphics theory]] [[maths]]
# Background and probability review

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
$$Pr\{x \in[a, b]\} = \int_{a}^{b}p(x)dx = P(b) - P(a)$$This follows the fundamental theorem of calculus.

Finally the **proabability mass function (PMF)** and **probability density function** are the same. The latter is for discrete values like, 1, 2, 3 and the other is for continuous values like all the real values over the domain of $[0, 1]$.
# References
##### Main Notes
[[Expected values]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]