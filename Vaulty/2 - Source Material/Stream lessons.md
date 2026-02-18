# Reference https://www.youtube.com/channel/UCWpkUgQIZLXg0dnvkoOg6OA

Uniform buffers and draw calls don't have a 1 to 1 relationship. Meaning that your draw calls that get recorded to a command buffer don't always match up with meaning it's not really possible to have a draw call per descriptor set unless you have different descriptor sets.

How if you want to have away to record things and update them you'll want to use push constants. 