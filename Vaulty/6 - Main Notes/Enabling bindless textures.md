2026-02-27 08:41
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]]
# Enabling bindless textures

This is actually pretty simply if you are working with vulkan 1.2+ because it became core in vulkan 1.2. If you are working with that you just need to work with the **VkPhysicalDeviceVulkan12Features**

```c++
VkPhysicalDeviceVulkan12Features physical12Features{};
physical12Features.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_VULKAN_1_2_FEATURES;
physical12Features.runtimeDescriptorArray = true;
physical12Features.descriptorBindingPartiallyBound = true;
```

You actually need 2 features for bindless textures. We need both of these feature because to get bindless textures working we need to bind the are array of textures first THEN updated them. This is different from regular descriptor sets.

```c++
VkPhysicalDeviceFeatures2 physicalDeviceFeature2{};
physicalDeviceFeature2.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_FEATURES_2;
physicalDeviceFeature2.pNext = physical12Features;
vkGetPhysicalDeviceFeatures2(vulkanPhysicalDevice, &physicalDeviceFeature2);
```

Make to actually enable this by building up the linked list for this feature.
# References
##### Main Notes
[[What is bindless]]
[[Creating the bindless descriptor pool]]
#### Source Notes
[[Bindless - Mastering Graphics Programming with Vulkan]]