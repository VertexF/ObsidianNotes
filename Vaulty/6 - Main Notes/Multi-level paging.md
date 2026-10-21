2026-09-11 09:53
Status: #baby 
Tags:[[OS]] [[OS memory]]
# Multi-level paging

Make sure you know about [[Introduction to paging]] before we can even begin here. The problem that multi-level paging is trying to solve is reducing memory wastage. It's very possible to have a linear page table set up that allocates way too many pages.

Introducing the multi-level paging which adds level of **page table directory** as levels of indirection.  We do this to keep the page tables/directories in 1 page.

We want our page table and page directories to fit within the size of a single page making page tables actually usable per-process. To start things off we need to know the **page director base register** value which will be loaded into a register inside the CPU's MMU when a process loads. 

If we have a hierarchy of page directories this actually reduces the amount of page entries we have in the final page table. Meaning that we can get more entries to fit with the **Translation Lookaside Buffer** (TLB) 

Then using the using the virtual address VPN and split it up into the **page directory index number** and **Table Entry Index** . We first get the  **page directory index number** which is the left most part of the virtual addresses. The totally bits that would be would be what the total bits to cover the page so if we had the unrealistically-small 32 bytes page size, we would need five bits because $2^{5} = 32$. 

If we wanted to find the **page table** from the **page directory**, we first need to get the **page directory base register**. The memory at the the page directory registers is just a bunch of values that point at page tables. So we need to get that index from that array. We need that with the `Page Table Index = PDBR + (PDIndex * sizeof(Page Size))`.

Next we check to see if  that value starts with a 1 it's value in binary if that value starts with a 0 then it's invalid. If it's valid we have to continue and repeat the process. Using the next part of the VPN from the virtual address we want to get the **page table entry**. We just do the same bit this time it's we get the page table memory, index into that following the next 5 bits because again $2^{5} = 32$. Index into that table entry and look up the **Physical Frame Number** (PFN) + the valid bits.

If the **Physical Frame Number** (PFN) starts with a 0 then it's invalid. Removing the invalid/valid bit we get the physical frame number. We take that value multiply it by 32 because the page sizes are 32 bytes in size and then add the offset from the virtual address which comes from the remaining bits of the virtual address. So `Physical Address = PFN * sizeof(Page size) + offset`.
# References
##### Main Notes
[[Introduction to paging]]
#### Source Notes
