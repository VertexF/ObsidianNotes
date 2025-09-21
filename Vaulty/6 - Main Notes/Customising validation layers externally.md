2025-09-21 10:40
Status: #adult 
Tags: [[vulkan]] [[vulkan validation]]
# Customising validation layers externally

You can configure the validation layers in different ways beyond the VkDebugUtilsMessengerCreateInfoEXT struct. If you go your installed vulkan SDK and look for the **Config** directory, you will find a file called **vk_layer_settings.txt** that file should explain how to configure things.

You'll need to do some project configuration to customise your validation layer usage. You will need to copy your layer settings to debug and/or release directories in your project once you've changed things. If you don't do that you'll be running with the debug set of validation layers, which is totally chill.
# References
##### Main Notes
[[What are validation layers]]
#### Source Notes
[[Vulkan-Tutorial]]