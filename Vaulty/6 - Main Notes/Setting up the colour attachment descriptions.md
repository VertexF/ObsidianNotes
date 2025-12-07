2025-10-30 18:58
Status: #baby 
Tags: [[vulkan]] [[render pass]]
# Setting up the colour attachment descriptions

The attachment description describes what needs to be changed within a buffer and samples within a render pass. So a colour attachment description is going to be different to a depth attachment description for example. Please note that the samples are different from a sampler.

You do this by setting creating the **VkAttachmentDescription**. I will show how to set up a simple colour attachment description but other types require different set up.

```c++
VkAttachmentDescription colourAttachment{};
colourAttachment.format = swapchainFormat;
colourAttachment.samples = VK_SAMPLE_COUNT_1_BIT;
```

I'm using the swapchain format to set the colour attachment format. The samples at set to **1_BIT** because we aren't doing any type of multisampling.

When creating attachment descriptions you need to describe how your doing to store and load the data. We do this with **.loadOp** and **.storeOp** for colour and depth data and **.stencilLoadOp** and **.stencilStoreOp** for stencil values only.

For **loadOp** we have these options:
- **VK_ATTACHMENT_LOAD_OP_LOAD** perserve the previous content of the attachment
- **VK_ATTACHMENT_LOAD_OP_CLEAR** clear everything back to a constant value
- **VK_ATTACHMENT_LOAD_OP_DONT_CARE** original content is undefined meaning we don't care about it.
For **storeOp** we have these options:
- **VK_ATTACHMENT_STORE_OP_STORE** we store rendered data in memory to read later.
- **VK_ATTACHMENT_STORE_OP_DONT_CARE** stuff in the framebuffer will be undefined after rendering operations.

For a standard colour attachment description you'll want to clear the framebuffer at the beginning of swapchain which will be the **loadOp** setting. For **storeOp** you'll want to store the result of the operation so we can see what's rendered.

We don't care about the stencil data because we're not using it.

```c++
colourAttachment.loadOp = VK_ATTACHMENT_LOAD_OP_CLEAR;
colourAttachment.storeOp = VK_ATTACHMENT_STORE_OP_STORE;
colourAttachment.stencilLoadOp = VK_ATTACHMENT_LOAD_OP_DONT_CARE;
colourAttachment.stencilStoreOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
```

Framebuffer's are just VkImage's. These images need to be transitioned from one memory format to another depending on the usage. The most common transition you have are:
- **VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL** image that need to be used as a colour attachment
- **VK_IMAGE_LAYOUT_PRESENT_SRC_KHR** images that need to be presented. 
- **VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL** images that need to be transfered with a copy operation.

You'll need to do a lot more sometimes to transition the image's in and out of different formats. For a standard colour attachment that hasn't be used you'll want this set up.

```c++
colourAttachment.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
colourAttachment.finalLayout = VK_IMAGE_LAYOUT_PRESENT_SRC_KHR;
```

**.initialLayout** is the state of the VkImage before the render pass begins. **.finalLayout** is the state of the VkImage after the render pass has finished. 

When you use **VK_IMAGE_LAYOUT_UNDEFINED** you are saying you don't care what the format is in before rendering. When a VkImage is in an UNDEFINED layout treat the VkImage as a unitialised memory and **NOT** cleared.

For all the render passes you end up using you'll be wanting to swap the attachments around.
# References
##### Main Notes
[[What is a render pass]]
#### Source Notes
[[Drawing a triangle]]
