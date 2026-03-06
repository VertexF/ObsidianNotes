# Reference https://www.youtube.com/watch?v=Ju4rXct6mPI

##### Pipeline state object
When you create a graphics pipeline and then run a draw command, the driver has to take that specific combinations of graphics pipeline and actually translate that to the hardware. Meaning that for the first frame things are slower and may make your stutter for awhile until the driver has cached all your combinations of graphics pipelines. This is what pipeline state objects are trying to solve.

When you create the pipeline state object it does all the GPU hardware set up when you create the object rather than at draw time.

There are two problems:
1) There is a lot of work you would have to do to get this working because everything from assert loading to everything just before the main loop needs to be aware that they maybe storing pipeline state stuff.
2) You also have combination explosion. In major games you would have tens of thousands vertex shader and fragment shaders. This happens because shaders end up doing way more than 1 thing.