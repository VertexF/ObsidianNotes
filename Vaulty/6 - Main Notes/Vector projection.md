2026-07-02 12:15
Status: #baby 
Tags: [[maths]]
# Vector projection

Sometimes we need to be able to work out the 2 vectors that make up 1 vector by decomposing them into seperate components. You can just get the length of the $x, y$ and $z$ axis to get the individual components of vectors.

It's important to note the basics of vector projection gets easier to understand when you work with basis vector. If you have basis vectors for a space $\vec{i}, \vec{j}$ and $\vec{k}$ we take the dot project of $(\vec{v} \cdot \vec{i})\vec{i}$ and the same for $j$ and $k$. This is case for when things are normalised. However to project properly we can use just the general equation as
$$a_{||b} = \frac{\vec{a} \cdot \vec{b}}{||b||^2} \vec{b}$$
Here we are project a vector from $\vec{a}$ onto $\vec{b}$ We are dividing by $b^2$ because $\vec{b}$ may not be a normal vector. If the dot product is negative meaning that angle $\theta$ between $\vec{a}$ and $\vec{b}$ is greater than 90 degree we get a negative projection, meaning the projection faces the other way.
# References
##### Main Notes
[[Vector rejection]]
[[Parametric lines]]
#### Source Notes
[[Foundations of Game Engine Developemt - Maths]]