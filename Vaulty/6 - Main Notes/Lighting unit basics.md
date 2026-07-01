2026-06-19 11:43
Status: #baby 
Tags: [[graphics theory]] [[light theory]]
# Lighting unit basics

It's a good to have lights that have coherent lighting units that all follows the same system. You have two different lighting types direct lighting and indirect lighthing.

| Photometric terms   | Notation   | Unit                             |
| ------------------- | ---------- | -------------------------------- |
| Luminous power      | $\Phi$     | Lumen($lm$)                      |
| Luminous intensity  | $I$        | Candela($cd$) or $\frac{lm}{sr}$ |
| Illuminance         | $E$        | Lux($lx$) or $\frac{lm}{m^{2}}$  |
| Lumiance            | $L$        | Nit ($nt$) or $\frac{cd}{m^2}$   |
| Radiant power       | $\Phi_{e}$ | Watt($W$)                        |
| Luminous efficacy   | $\eta$     | Lumens per watt ($\frac{lm}{W}$) |
| Luminous efficiency | $V$        | Percentage $(\%)$                |
Illuminance is the total liminous flux incident on a surface per unit area. That is why we mesure this with a $\frac{lm}{m^{2}}$ because it's per unit area.

Candela $cd$ is a standard unit that cares about the luminous power over a solid angle from a given light. A solid angle can be thought of as a cone were the apex of the cone starts at the light source.

![[Angle_solide_coordonnees.svg.png]]

A solid angle is expressed in a dimensionless steradian an $sr$ unit. Hence the $cd = \frac{lm}{sr}$ for working our the luminous power over a solid angle. If we were to measure solid angles we would use steradians. Think about steradians like a steradian to a sphere what a radian is to a circle.

Lumiance is a measure of the luminous intensity per unit area of light travelling in a given direction using a solid angle. That's also why we are mesuring $\frac{cd}{m^2}$ per unit area vs travelled over a solid angle.

So it's good idea to make sure your lights map to real world light units. Here table of common lights of units you should try to use. 

| Light type             | Unit                                   |
| ---------------------- | -------------------------------------- |
| Directional light      | Illuminance ($lx$ or $\frac{lm}{m^2}$) |
| Point light            | Luminous power ($lm$)                  |
| Spot                   | Luminous power ($lm$)                  |
| Photometric light      | Luminous intensity ($cd$)              |
| Mask photometric light | Luminous power ($lm$)                  |
| Area light             | Luminous power ($lm$)                  |
| Image based light      | Luminance ($\frac{cd}{m^2}$)           |
So if want to use watts to work with light brightness you can use this formular to go from lumen over watts to just lumens. Artist for some reason like to use watts.
$$\Phi = \Phi_e \eta = W \frac{lm}{W}$$
So $\eta$ is luminous efficacy of the light expressed in lumens per watt. The maximum possible luminous efficacy is $683\frac{lm}{W}$ We can also just use $V$ also called luminous coefficient. As this is the power using the number of photons per watt.
$$\Phi = \Phi_e 683 \times V$$
Look online to learn specific lighting luminous efficacy or the luminous efficiency if you're using a coherent lighting system.
# References
##### Main Notes
[[Light distribution]]
[[Light unit measuring basics]]
[[The point light]]
[[The spot light]]
[[The directional light]]
[[Direct lighting theory]]
[[Attenuation function]]
#### Source Notes
[[Filament PBR - Lighting]]