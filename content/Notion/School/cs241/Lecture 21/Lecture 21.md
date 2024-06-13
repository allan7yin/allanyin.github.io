Previously, we looked at the issue of fragmentation. This occurs, when we allocated memory multiple times, then free in some order, where we are left with certain disjoint blocks of memory. Consider:

![[Screenshot_2023-04-23_at_10.26.52_PM.png]]

To reduce fragmentation, there are 3 allocation strategies we can follow for variable-sized blocks:

1. **First Fit:** find the first hole the size fits in, and allocated it. This works well in practice and is fast
2. **Best Fit:** find the location that has the least amount of leftover space
3. **Worse Fit:** find the biggest hole, so that a relatively large hole remains after allocating, which can then satisfy other requests
    1. This has the most fragmentation, least utilized in practice

  

So that was variable-sized allocation. Now, we look at the 4th approach: **Binned Block Sizes**. This is kind of like variable block sizes, but instead of being any size, there are certain sizes to choose from, which makes managing the blocks easier. 2 methods:

1. **dlmalloc**
2. **binary buddy system**

  

**⇒ dlmalloc**

For this approach, we split the allocations into 3 categories:

- small ones (**smallbin**): which are **512 byes or less**
- medium ones: **512 bytes to 256 KB**
- large ones: **greater than 256 KB**

  

**Smallbin** requests have bins of various sizes: multiples of 16 starting at 32 → 32, 64, 80, … 512. To implement this, we have multiple free lists for holes of different sizes. This is for smallbins, for medium and large ones, we will need more complex data structures like tries.