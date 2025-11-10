2025-11-09 15:12
Status: #baby 
Tags: [[Vulkan]] [[vulkan swapchain]]
# Recreating the swapchain

Note I've coded things slightly differently from the Vulkan-Tutorial.

If you want to resize the window surface changes which means the swapchain and surface aren't compatible anymore. When this happens you have to recreate the swapchain.

To recreate the swapchain you need the swapchain, image views created from the swapchain images and the framebuffer created from the swapchain images.

It's important to note that you might want to even create the renderpass. It's possible that a monitor has lower DPI and then the window gets dragged to a higher DPI monitor. For now I will skip that.

```c++
static void recreateSwapchain()
{
    vkDeviceWaitIdle(device);

    vkGetPhysicalDeviceSurfaceCapabilitiesKHR(physicalDevice, surface, &surfaceCapabilities);
    swapchainExtent = surfaceCapabilities.currentExtent;

    if(swapchainExtent.width == 0 || swapchainExtent.height == 0) 
    {
        return;
    }

    cleanupSwapchain();

    createSwapchain();
    createImageViews();
    createFramebuffers();
}
```

First thing we need to do is wait for the GPU with **vkDeviceWaitIdle(device);** You can't change resources that are still in use so when we are recreating the swapchain we have to pause the GPU to before we can continue.

The next thing we do is guard against the viewport degrading to 0 height or width either through a mouse drag or the Window being minimised. When recreating the swapchain the **.imageExtent** have to be greater than 0 for width or hieght in the **VkSwapchainCreateInfoKHR** struct. To test this we query the **surfaceCapabilities** which will be used in the **createSwapchain();** function if we aren't at 0 height or width.

Next we have to destroy the old framebuffers, old ImageView's and the framebuffers, all before we start recreating them. This is done the **cleanupSwapchain();**

```c++
static void cleanupSwapchain() 
{
    for (auto framebuffer : swapchainFramebuffers)
    {
        vkDestroyFramebuffer(device, framebuffer, nullptr);
    }

    for (auto imageView : swapchainImageViews)
    {
        vkDestroyImageView(device, imageView, nullptr);
    }

    vkDestroySwapchainKHR(device, swapchain, nullptr);
}
```

Creating the swapchain happens in the **createSwapchain();** function. Below is everything you need to write that function.
[[What is a swapchain]]
[[Checking surface and swapchain compability]]
[[Setting up the swapchain extension]]
[[Querying surface capabilities]]
[[Checking surface format]]
[[Selecting a presentation mode]]
[[Creating the swapchain]]
[[Getting swapchain images]]

Creation of the swapchain image views happens in the **createImageViews();** function. Below is everything you need to write that function.
[[What are image views]]
[[Creating an image view]]
[[Destroying a image view]]
 
Creation of the framebuffers happens in the **createFramebuffers();** function. Below is everything you need to write that function.
[[What is a framebuffer]]
[[Creating the framebuffers]]

You can do better than this because we aren't going to use the **.oldSwapchain** feature when creating of the new swapchain. This allows us to still deal with frames in flight while the swapchain is being recreated. 
# References
##### Main Notes
[[When to recreate the swapchain image]]
**Swapchain**
[[What is a swapchain]]
[[Checking surface and swapchain compability]]
[[Setting up the swapchain extension]]
[[Querying surface capabilities]]
[[Checking surface format]]
[[Selecting a presentation mode]]
[[Creating the swapchain]]
[[Getting swapchain images]]
**Image View**
[[What are image views]]
[[Creating an image view]]
[[Destroying a image view]]
**Framebuffer**
[[What is a framebuffer]]
[[Creating the framebuffers]]
#### Source Notes
[[Drawing a triangle]]