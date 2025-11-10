2025-09-21 08:28
Status: #adult  
Tags: [[vulkan]] [[vulkan validation]]
# Why set up debug callbacks for vulkan validation

This extensions requires a bit of effort to set up but it's worth the effort. The reason being is that by default the validation layers will just simply prints things to standard output stream. 

With your own custom callback function you can filter messages by severity and type meaning you have tight control on what you consider an error. You can add a forced break point when the callback function runs. You can even pass back a pointer to an object you might want to use for logging or to simply look at.
# References
##### Main Notes
[[What are validation layers]]
[[Creating the custom callback function for validation]]
[[Connecting custom debug function to Vulkan]]
#### Source Notes
[[Drawing a triangle]]