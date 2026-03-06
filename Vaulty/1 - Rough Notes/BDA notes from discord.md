
do you know what data a buffer descriptor contains?
a pointer in gpu memory, and a size this is in constrast to image descriptors, which have a lot of data 64 bytes on AMD, dunno on nvidia.
the size is only used by the robustness extensions anyways
the hardware just reads data using the pointer
BDA lets you access that pointer for yourself
and exposes the hardware's ability to read data using any pointer
you can do pointer arithmetic and stuff just like you can on the cpu
even build a linked list or something lol