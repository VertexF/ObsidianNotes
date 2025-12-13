2025-10-26 09:33
Status: #baby 
Tags: [[vulkan]] [[vulkan graphic pipeline]]
# Setting up the rasteriser

The rasteriser takes 3D and 2D geometry and turns it into fragments to be used by the fragment shader. This is more than just pixel, it also contains things like fragment position, a depth and stencil value and alpha. It's possible for it to contain things like the window id. These details are all hidden from us for easy of use.

That being said this is were the depth testing, face culling and scissor tests are ran. You set this up with the struct **VkPipelineRasterizationStateCreateInfo**.

```c++
VkPipelineRasterizationStateCreateInfo rasteriser{};
rasteriser.sType = VK_STRUCTURE_TYPE_PIPELINE_RASTERIZATION_STATE_CREATE_INFO;
rasteriser.depthClampEnable = VK_FALSE;
```

If you set the **.depthClampEnable** to VK_TRUE you'll clamp the fragments that a beyond the near and far planes rather than discarding them. This might be useful if you're doing shadow mapping, but this requires a GPU feature to be turned on.

```c++
rasteriser.rasterizerDiscardEnable = VK_FALSE;
```

If this is set to VK_TRUE everything gets thrown out. This is useful to set to true if you're debugging how long the pre-fragment shader part of the pipeline takes.

```c++
rasteriser.polygonMode = VK_POLYGON_MODE_FILL;
```

Here you have 3 different setting you could use
- VK_POLYGONE_MODE_FILL - This is the normal fille the entire polygon.
- VK_POLYGONE_MODE_LINE - This will only save the lines between each of the vertices.
- VK_POLYGONE_MODE_POINT - This will only render the vertices as points.
Any other feature you find in vulkan requires a GPU feature to be enabled.

```c++
rasteriser.lineWidth = 1.f;
```

This will set the line width the rasteriser will output. Anything more than 1.f requires the GPU feature **wideLines** to be enabled.
#### NOTE
If you haven't set up a view projection matrix you want to set things up like this

```c++
rasteriser.cullMode = VK_CULL_MODE_BACK_BIT;
rasteriser.frontFace = VK_FRONT_FACE_CLOCKWISE;
```

These set up the culling modes. **.cullMode** tells vulkan what cull if anything at all. **.frontFrace** tells vulkan what direction the polygons are facing.
#### NOTE
If you have set up the view projection matrix your cull mode needs to be **VK_FRONT_FACE_COUNTER_CLOCKWISE** because you'll have to flip the Y direction.

```c++
rasteriser.frontFace = VK_FRONT_FACE_COUNTER_CLOCKWISE;
```

If you don't do this the back face culling will kick and remove the polygon from view.

# References
##### Main Notes
#### Source Notes
[[Drawing a triangle]]