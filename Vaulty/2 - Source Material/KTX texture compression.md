# Reference https://www.khronos.org/ktx/
https://docs.vulkan.org/samples/latest/samples/performance/texture_compression_basisu/README.html
##### KTX: GPU Texture Container Format
Basis universal superspressed GPU texture are only on ktx2. This tech allows produces compact textures that can be transcoded to variety of GPU compressed formats at run time.

The GLTF extension **KHR_texture_basisu** means that the GLTF model contains KTX 2.0 textures.

![[Universal-GPU-Compressed-Textures.png]]

So important to note you should be transcoding the textures into a GPU compressed format, such as BC7 or whatever.
##### gltfpack compression settings
When using `gltfpack` to compress your textures there are two options `-tc` which seems to compress the textures with ETC1S compression which is smaller lower quality version. As far as can see this compression look fine. The `-tu` compresses textures with UASTC which is the higher quality larger file format.

Without these options you don't compress any of the textures when simply running gltfpack into ktx container.
##### KTX Container Format
So when you are trying to support compressed textures there are three format concepts you need to be aware of.
1) **Container format** You need KTX wrap the transmission format data. This is just the meta about a particular image, such as dimensions, compression type, how to transcode that data into a GPU-compatible pixel format.
2) **Transmission format** This is the highly compressed representation of pixel data layed out in a way that can be easily transcoded to one or more GPU compressed formats. Like **ETC1S** or **UASTC**
3) **GPU compressed pixel format** A compressed representation of pixel data unstood by the GPU. Such as **BCn**, **ASTC**, **ETC** and **PVRTC1**.

You can read, write and transcode `.ktx2` stuff with the with [KTX-Software](https://github.com/KhronosGroup/KTX-Software/): Includes [downloadable binary packages](https://github.com/KhronosGroup/KTX-Software/releases) and WebAssembly builds.
##### Block-based compression
To support random access, compressed textures are normally organised into blocks of the same size. Basis unverisal always used blocks of 4x4.

To estimate the GPU memory required for a given texture you should use the following expression.

```c++
uint32_t widthOfBlock = (WIDTH_IN_PIXELS + 3) >> 2;
uint32_t heightOfBlock = (HEIGHT_IN_PIXELS + 3) >> 2;

uint32_t blockCount = widthOfBlock * heightOfBlock;
```

Applications should expection mip levels to be 1, 2 or multiple of 4 and if that's the case you should reject the texture.

The dimensions of subquent mip levels follows the truncating division by 2 rule.

| Mip Level | Width x Height, px | Width x Height, blocks |
| --------- | ------------------ | ---------------------- |
| 0         | 100 x 200          | 25 x 50                |
| 1         | 50 x 100           | 13 x 25                |
| 2         | 25 x 50            | 7 x 13                 |
| 3         | 12 x 25            | 3 x 7                  |
| 4         | 6 x 12             | 2 x 3                  |
| 5         | 3 x 6              | 1 x 2                  |
| 6         | 1 x 3              | 1 x 1                  |
| 7         | 1 x 1              | 1 x 1                  |
You can have a mip levels like`25x50` which fill up an entire compressed block but that's not always the case. Sometimes you need to pad data but this pixels cannot be access by sampling.
##### ETC1S / BasisLZ Codec
ETC1S/BasisLZ is a hybrid compression schme that rearranges ECT1S texture block data with some LZ-style lossless compression. It priorities luma information which gets a very storage/transmission efficiency for things like colours textures. It's not as good for things like normals maps but I'm sure it's fine.

After decoding the LZ compression, you can losslessly repack into ETC1 texture blocks or transcode into another block-compressed format.
##### Data layout for ETC1S
ETC1S repesents a subset of ETC1 so compressed always has three colour channels internally. To support textures other than opaque colour texture ETC1S might contain an extra slice. So this means that ETC1S data format descriptor in KTX2 container has one or two different `channelType`'s

| Channels  | First Slice                | Second Slice               | Typical Usage                     |
| --------- | -------------------------- | -------------------------- | --------------------------------- |
| RGB       | `KHR_DF_CHANNEL_ETC1S_RGB` | Not present                | Opaque colour texture             |
| RGBA      | `KHR_DF_CHANNEL_ETC1S_RGB` | `KHR_DF_CHANNEL_ETC1S_AAA` | Colour texture with alpha channel |
| Red       | `KHR_DF_CHANNEL_ETC1S_RRR` | Not Present                | Single-channel texture            |
| Red-Green | `KHR_DF_CHANNEL_ETC1S_RRR` | `KHR_DF_CHANNEL_ETC1S_GGG` | Dual-channel texture              |

##### Runtime usage
Using ETC1S/BasisLZ data is made of three 3 steps
1) First thing you need to initialise the transcoder with data common to all part of the textures, such texture face, arrays and mip levels.
2) Calling the coder with per-slice data and the texture target format needed.
3) Uploading transcoded data to the GPU.

When the texture data uses a non-linear sRBG encoding (which is virtually true for all textures), the application should use the hardware decoder to move to sRGB. You just need to correctly select the right enum with proper texture format enum.

Note that single red and red-green don't support sRGB encoding anyway.
##### Transcode target selection for RGB and RGBA
By design the ETC1S is a strict subset of ETC1 so transcoding it to an ETC format is always preferred. 