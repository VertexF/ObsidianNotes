2025-12-06 19:33
Status: #baby 
Tags: [[vulkan]] [[depth testing]]
# Setting up reverse perspective for your depth buffer

The theory can be found here [[What is reverse-Z perspective]]

You do need to do things differently for
- [[Setting up the depth and stencil testing]]
- [[Clear depth values]]

Firstly we need to make sure that we use the right perspective in our cglm calculation. You will want to define the macro 
```c++
#define CGLM_FORCE_DEPTH_ZERO_TO_ONE
#include <cglm/struct/vec2.h>
#include <cglm/struct/vec3.h>
#include <cglm/struct/affine.h>
#include <cglm/struct/cam.h>
```

This begin things off right with having our range from 0 to 1, if you don't define this the range is -1, 1 which we aren't interested in. 

Next you'll need to make the perspective matrix with the opposite far planes and near plane values.

```c++
glms_perspective(glm_rad(45.f), swapchainExtent.width / (float)swapchainExtent.height, 100.f, 1.f);
```

Here **100.f** is the near plane and **1.f** is the far plane. Think about what we said before, the depth value is worked out to basically be 1/z and z is the world space depth. If we reverse the z values then 1/z which works out the depth value is reversed which is what we want.

Next in the graphics pipeline you need to change what the **.depthCompareOp** tests are.
```c++
depthStencil.depthCompareOp = VK_COMPARE_OP_GREATER_OR_EQUAL;
```

Why is the **VK_COMPARE_OP_GREATER_OR_EQUAL**? Well remember our Z values are reversed so it's actually the larger values that are closer to the near plane. So we want to keep fragments that are that are larger as they are closer to the near plane now. It's also true that **VK_COMPARE_OP_GREATER_OR_EQUAL** is the opposite to **VK_COMPARE_OP_LESS** so just by that alone it tells us it's want we want.

Finally we need to clear the depth values. Remember for the clear depth value the we start everything at the far plane so nothing get initially rejected. Well now it need to 0 and not 1. Since 0 is at the far plane now.

```c++
std::array<VkClearValue, 2> clearColour{};
clearColour[0] = { { 0.f, 0.535f, 1.f, 1.f } };
clearColour[1] = { { 0.f, 0 } };
renderPassBeginInfo.clearValueCount = uint32_t(clearColour.size());
renderPassBeginInfo.pClearValues = clearColour.data();
```

If you do that everything so render like you're in normal perspective but now you have a lot more depth values to work with between the far and near plane.
# References
##### Main Notes
[[Setting up the depth and stencil testing]]
[[Clear depth values]]
#### Source Notes
[[Depth buffering]]