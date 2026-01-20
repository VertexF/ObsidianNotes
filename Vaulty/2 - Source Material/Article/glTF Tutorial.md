# Reference https://github.khronos.org/glTF-Tutorials/gltfTutorial/

##### Introduction
The whole point of GLTF files is that we have a file format that's built for real-time applications. Which are application with a time limit on execution. If you get a different file format you might be able to move it to GLTF with the open source tools https://github.khronos.org/glTF-Project-Explorer/ That's if blender doesn't work.
##### Basic GLTF structure
In a GLTF file you can have more than one model, this is what the scene is there for to do, it describes more than just a mesh.

If you want to access things in the GLTF file you can access them by their index in JSON arrays.
```json
"meshes" : 
[
    { ... }
    { ... }
],
```

In the arrays there are indices of the array in JSON have a key value pair that points back to objects to build a hierarchal relationship between objects and the array of objects.

```json
"nodes": 
[
    { "mesh": 0, ... },
    { "mesh": 5, ... },
    ...
],
```

So as you can see mesh 0 and mesh 5 need to have whatever is in these nodes.

This is the relationship layed out. 
![[gltfJsonStructure.png]]

- **scene** is the top level node which has an array of nodes.
- **node** the nodes have have 5 different things, mesh, camera, skin, child node and transform objects.
- **camera** the camera set up a camera configuration which is basically useless for use.
- **mesh** the describes the geometry of what's in the scene. It has accessors which are were the actual geometry lives and materials is just that.
- **skin** this holds the information vertex skinning to deform a character. The values of these parameters are obtained from an `accessor`.
- **animation** tells us how the transformation change over time for certain nodes.
- **accessor** these contain the data for the geometry data for the mesh, skinning and the skinning parameters and the time-dependent animation values for animation.
- ****