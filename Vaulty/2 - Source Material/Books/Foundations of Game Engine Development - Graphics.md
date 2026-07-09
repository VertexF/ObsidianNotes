# Reference file:///D:/Shared/Books/(Foundations%20of%20Game%20Engine%20Development%202)%20Eric%20Lengyel%20-%20Foundations%20of%20Game%20Engine%20Development%20Volume%202%20Rendering.%202-Terathon%20Software%20(2019).pdf

##### Billboards
So there 2 types of billboards spherical and cylindrical this is nothing to do with the geometry but how it's oritentation works relative to the camera.

You can also get a polyboard which are used for things like smoke particles as they are like billboards all stuck together.

Polygon trimming is to do with optimisations for billboards with large transparent regions, useful in such as tree impostors.
##### Spherical billboards
Spherical billboard is defined by a centre point $p$ and two radii $r_{x}$ and $r_{y}$ equal to the half width and height. The billboard can rotate about orientation but it's normal vector needs to always face the camera. So the union of all possible orientations is a sphere of radius $\sqrt{r^2_{x} + r^2_{y}}$ 

We need to move the billboard vertices into billboard object space. Lets set this up $M_{c}$ which will be the transform to camera space to world space, and $M_{o}$ which will be the transform from billboard object to world space.

To connect both spaced $M_{c}$ and $M_{o}$ together to make that the tranformations work, you want to get both vectors $\vec{x}$ and $\vec{y}$ be the postive $x$ and $y$ axes of the camera given by
$$\vec{x} = M^{-1}_{o}M_{c}[0]$$

$$\vec{y} = M^{-1}_{o}M_{c}[1]$$
Where $M_{c}[j]$ denotes the column $j$ of the matrix $M_{c}$ If we place vertices of a billboard in the plane determined by a point $p$ and directions of $\vec{x}$ and $\vec{y}$ then the resulting polygon is always parallel to the projection plan. 

To get the column vectors out of the view matrix you would want to do something like this.
```c++
    vec3 cameraRightWorld = { scene2D.sceneData2D.view[0][0], scene2D.sceneData2D.view[1][0], scene2D.sceneData2D.view[2][0] };
    vec3 cameraUpWorld = { scene2D.sceneData2D.view[0][1], scene2D.sceneData2D.view[1][1], scene2D.sceneData2D.view[2][1] };
```

If we have four vertex position $v_{0}$, $v_{1}$, $v_{2}$ and $v_{3}$ are given by.
$$v_{0} = p - r_{x}\vec{x} + r_{y}\vec{y} \space \space \space \space \space v_{1} = p + r_{x}\vec{x} + r_{y}\vec{y}$$
$$v_{2} = p + r_{x}\vec{x} - r_{y}\vec{y} \space \space \space \space \space v_{3} = p - r_{x}\vec{x} - r_{y}\vec{y}$$
If $M_{o}$ contains a scale then $\vec{x}$ and $\vec{y}$ these need to be normalised. For objects made of a single billboard. 

For a single billboard it is often the case that $p$ lies at the object-space origin. This means that the vertex positions are simply a sums of differences of $r_{x}\vec{x}$ and $r_{y}\vec{y}$. This doesn't work for particle systems as that contains a large number of independent billboards, there is different point $p$ representing the centre of each particle.
$$v_{0} = p - r_{x}\vec{x} + r_{y}\vec{y} \space \space \space \space \space v_{1} = p + r_{x}\vec{x} + r_{y}\vec{y}$$
$$v_{2} = p + r_{x}\vec{x} - r_{y}\vec{y} \space \space \space \space \space v_{3} = p - r_{x}\vec{x} - r_{y}\vec{y}$$
These equations only produce billboards that face only the x-y plane of the camera and not actual positions of the camera expect in those cases when the billboard lies at the centre of the viewport. This is actually mostly good enough from a visual standpoint, especially when the billboards are small in size. 

This is exactly what's happening here with picture a)
![[billboard-2.png]]
This is what it would look like in code
```c
const vec3 pos[4] = vec3[4]
(
	vec3(0.0, 0.0, 0.0),
	vec3(1.0, 0.0, 0.0),
	vec3(1.0, 1.0, 0.0),
	vec3(0.0, 1.0, 0.0)
);

const int indices[6] = int[6]
(
	0, 1, 2, 2, 3, 0
);

//This is just the input position of the billboard.
location(layout = 0) in vec3 vPosition;
//This is the size of the billboard.
location(layout = 1) in vec3 vRadius;

void main()
{
    int idx = indices[gl_VertexIndex];
    vec3 position = pos[idx];

    vec3 cameraRightWorld = { scene2D.sceneData2D.view[0][0], scene2D.sceneData2D.view[1][0], scene2D.sceneData2D.view[2][0] };
    
    vec3 cameraUpWorld = { scene2D.sceneData2D.view[0][1], scene2D.sceneData2D.view[1][1], scene2D.sceneData2D.view[2][1] };

    
    vec3 positionWorld = vPosition + 
					     vRadius * position.x * cameraRightWorld +
						 vRadius * position.y * cameraUpWorld;

    gl_Position = scene2D.sceneData2D.project * scene2D.sceneData2D.view * vec4(positionWorld, 1.0);
    
}
```
##### A more accurate spherical billboard 
If you want things to be more accurate you need to do more work. 