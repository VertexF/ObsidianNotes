2026-02-11 11:28
Status: #baby 
Tags: [[performance]] [[DOD]]
# Aligning your structs for better performances

What you would is make sure that your padding your structs on the CPU side to have good alignment so they can be more efficiently pulled out of memory. 

You can do this by these three major ideas. 
1) [[Reducing struct memory alignment|Reducing struct memory alignment if possible]]
2) [[Storing bools out of bounds|Reduce the amount of member variables.]]
3) [[Moving data to helps alignment|Put the largest data variables at the top of struct and smallest at the bottom.]]
##### Main Notes
[[What is memory alignment]]
[[Reducing struct memory alignment]]
[[Storing bools out of bounds]]
[[Moving data to helps alignment]]
#### Source Notes
[[Intro to Data Oriented Design for Games]]