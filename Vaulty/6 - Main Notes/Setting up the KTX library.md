2026-07-25 11:16
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]]
# Setting up the KTX library

Setting up this library was difficult so I want to document how I did. The first set is to download the release on github [KTX 4.4.2](https://github.com/KhronosGroup/KTX-Software/releases/tag/v4.4.2) This is the most up to date version of KTX that's official release. 

The reason you want to do this is that library is that set up is differently making it harder to set up. Setting up KTX2 capabilities requires of external includes verse version 1.

```cmake
set(DF_INCLUDE "${CMAKE_CURRENT_SOURCE_DIR}/vender/KTX-Software-4.4.2/external/dfdutils/KHR")
set(KTX_LIB_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/vender/KTX-Software-4.4.2/include")
set(KTX_LIB_UTILS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/vender/KTX-Software-4.4.2/utils")
set(KTX_STDZ "${CMAKE_CURRENT_SOURCE_DIR}/vender/KTX-Software-4.4.2/external/basisu/zstd")
```

You need all 4 for the ktx library to work. You just need 1 `KTX_LIB_INCLUDE_DIR` to link with you main project.

To build the library you need all these files as part of the library.
```cmake
set(KTX_SOURCES
    vender/KTX-Software-4.4.2/external/basisu/zstd/zstd.c
    vender/KTX-Software-4.4.2/external/dfdutils/interpretdfd.c
    vender/KTX-Software-4.4.2/external/dfdutils/vk2dfd.c
    vender/KTX-Software-4.4.2/external/dfdutils/createdfd.c
    vender/KTX-Software-4.4.2/external/dfdutils/queries.c
    vender/KTX-Software-4.4.2/lib/texture.c
    vender/KTX-Software-4.4.2/lib/miniz_wrapper.cpp
    vender/KTX-Software-4.4.2/lib/vk_funcs.c
    vender/KTX-Software-4.4.2/lib/vkformat_check.c
    vender/KTX-Software-4.4.2/lib/vkformat_typesize.c
    vender/KTX-Software-4.4.2/lib/vkformat_check_variant.c
    vender/KTX-Software-4.4.2/lib/texture2.c
    vender/KTX-Software-4.4.2/lib/texture1.c
    vender/KTX-Software-4.4.2/lib/hashlist.c
    vender/KTX-Software-4.4.2/lib/checkheader.c
    vender/KTX-Software-4.4.2/lib/swap.c
    vender/KTX-Software-4.4.2/lib/memstream.c
    vender/KTX-Software-4.4.2/lib/filestream.c
    vender/KTX-Software-4.4.2/lib/vkloader.c)

add_library(ktx STATIC ${KTX_SOURCES})

target_include_directories(ktx SYSTEM PRIVATE ${KTX_LIB_INCLUDE_DIR}
                           ${KTX_OTHER_INCLUDE_DIR}
                           ${KTX_LIB_UTILS_DIR}
                           ${VULKAN_INCLUDE_DIR}
                           ${KTX_STDZ}
                           ${KTX_EXTERNAL}
                           ${DF_INCLUDE})
```

The `${VULKAN_INCLUDE_DIR}` is just the environment path include for vulkan.

Then you just link the library with your project.
```cmake
target_link_libraries(Void PRIVATE Foundation External Jolt ktx)
```

You need to turn KTX into a static library with the define. You can do this in CMake of course. 
```c++
#define KHRONOS_STATIC
#include <ktx.h>
#include <ktxvulkan.h>
```

NOTE: Updates the KTX library might break this and cause an issue. Also with more features you want to use you will need to add more `.c` files into the library. 
# References
##### Main Notes
[[What is KTX]]
[[Creating a texture with KTX file]]
#### Source Notes
