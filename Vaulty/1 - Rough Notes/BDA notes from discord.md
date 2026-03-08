
do you know what data a buffer descriptor contains?
a pointer in gpu memory, and a size this is in constrast to image descriptors, which have a lot of data 64 bytes on AMD, dunno on nvidia.
the size is only used by the robustness extensions anyways
the hardware just reads data using the pointer
BDA lets you access that pointer for yourself
and exposes the hardware's ability to read data using any pointer
you can do pointer arithmetic and stuff just like you can on the cpu
even build a linked list or something lol

I decided to look into how cubemaps work again
### A Short Note on Vulkan Cubemaps & Cubemap Arrays,
```c++
VkImageCreateInfo::flags        = VK_IMAGE_CREATE_CUBE_COMPATIBLE_BIT;
VKImageCreateInfo::imageType    = VK_IMAGE_TYPE_2D;              
VkImageCreateInfo::extent.width = VkImageCreateInfo::extent.height;
VkImageCreateInfo::extent.depth = 1;
VkImageCreateInfo::arrayLayers  = 6k;

VkImageViewCreateInfo::viewType                     = k == 1 ? VK_IMAGE_VIEW_TYPE_CUBE : VK_IMAGE_VIEW_TYPE_CUBE_ARRAY;
VKImageViewCreateInfo::subresourceRange::layerCount = VkImageCreateInfo::arrayLayers;
```
##### Notes:
1) k is a positive integer, k > 0. It represents the number of cubemaps.
2) VkImageViewCreateInfo::subresourceRange::layerCount MUST be equal to VkImageCreateInfo::arrayLayers, which is 6k. This might seem incorrect, but it isn't. See VUID-VkImageViewCreateInfo-viewType-02960, VUID-VkImageViewCreateInfo-viewType-02961, VUID-VkImageViewCreateInfo-viewType-02962 and VUID-VkImageViewCreateInfo-viewType-02963 for more information.
3) To create image views of type VK_IMAGE_VIEW_TYPE_CUBE_ARRAY, the Vulkan 1.0 feature imageCubeArray MUST be enabled.
4) Image array layers map to cubemap faces in the order X+, X-, Y+, Y-, Z+, Z-, in a right-handed Y-down coordinate system, as described in the specification.