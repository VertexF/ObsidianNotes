2026-07-02 11:12
Status: #baby 
Tags: [[maths]]
# The problem with transform normal vectors

Tangent vectors aren't affected by a matrix transforms because they are caculated based on the vertices of a model. Meaning that when $p_{1}$ and $p_{2}$ are tranformed by matrix $M$ then the tangent vector $t$ is just $Mt$. However, normal vectors can't be transformed in the same way.

If we scale a model non-uniformaly this scales normal vectors in away that destroy perpendicularity.
# References
##### Main Notes
[[Normal vector maths]]
[[Handling normal vectors when transformed]]
[[The adjungate normal matrix]]
#### Source Notes
[[Foundations of Game Engine Developemt - Maths]]