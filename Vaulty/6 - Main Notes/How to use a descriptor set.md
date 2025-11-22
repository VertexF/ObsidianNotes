2025-11-21 18:59
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]]
# How to use a descriptor set

To use descriptor set you need to specify the [[Creating a descriptor set layout|descriptor set layout]] and [[Creating uniform buffers|create buffers]] or images to use within the descriptor sets. 

You then need to make a [[Creating a descriptor pool|descriptor pool]], create the [[Creating descriptor sets|descriptor sets]] from that pool, update them by writing the data you want the descriptor sets to handle and finally [[Binding a descriptor set to use at draw time|bind]] that bad boy to run along side your draw call.
# References
##### Main Notes
[[Creating a descriptor pool]]
[[Creating descriptor sets]]
[[Binding a descriptor set to use at draw time]]
[[Creating a descriptor set layout]]
[[Creating uniform buffers]]
#### Source Notes
[[Uniform buffers]]