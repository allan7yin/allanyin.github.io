This lecture served more as an overview of the entire course. One of the key ideas present throughout the lecture, and likely throughout the course, is relationships between computer hardware and software. To begin becoming more interested/open with the idea of this, consider the following example:

```C++
\#include<stdio.h> 
\#define NR 10000 
\#define NC 10000 

int a[NR][NC];

void main() {
	int i,j;
	for (i=0;i<NR;i++) {
		for (j=0;j<NC;j++){ 
			a[i][j]=32767; 
		} 
	} 
} 
```

This is a very simple small snippet of code, that defines a 1000 by 1000 matrix, and assigns each index, the value of 32767. 32767 is also the highest number that can be represented in a **signed 16-bit integer**. There are two things we want to note here

1. The order that we access elements of the arrays make a difference in determining how long it takes to run the code
2. The operating system also has an impact.

Below are some benchmarks when we run this code snippet on two different machines:

> Raspberry Pi 400 Core A72 @ 1.8GHz  
> Row-by-row (  
> `a[i][j]`): 1.432 seconds  
> By column (  
> `a[j][i]`): 20.452 seconds

> Intel x86, Core i5 2.6GHz  
> Row-by-row (  
> `a[i][j]`): 0.484 seconds  
> By column (  
> `a[j][i]`): 1.583 seconds

  

Now, its very noticeable that there are performance differences. Mot notably, look how big of a change there is by simply swapping whether we access `i` or `j` first. Why is this the case? That is, why is accessing `i` first much quicker? This is because memory access is being fetched in blocks of 64 bytes (8 words at a time, with a word being 4 bytes for 32-bit architecture, and 8 bytes for 64-bit architecture). So, when accessing by `i` first ( as we iterate through `i` in the inner loop), time can be saved. On the other hand, by accessing `j` first, all those extra memory read in is now wasted, as move onto the next column. This data is not remembered, and will be needed to be re-read in the next time we visit this row.

  

In this course, we’ll look at ARM assembly, and in particular, a small subset of this, as ARM itself is too large to learn in this course alone. So, what is an assembler? Look at this hierarchy:

![[Untitled 9.png|Untitled 9.png]]

  

![[Screenshot_2023-01-11_at_9.30.18_PM.png]]

We know that compiler compiles a high-level language, such as C++, into machine code. Assembly is a 1:1 symbolization of machine code.

  

Assembly is a low-level language designed to simplify the instructions fed into a computer’s CPU, in other words, it’s a human readable abstraction on top of machine code so programmers don’t have to manually count one’s and zero’s. This is is what **ARM** is, an assembly language, and used in things like Apple M1, and Raspberry Pi. **x86** is used for intel chips.

![[Screenshot_2023-01-11_at_9.41.57_PM.png]]

---

# Arm Overview

Please read the below, to learn more about **ARM**. As **ARM** is much to large to use in CS251, we’ll at a smaller subset, **LEGv8**. Here is a set of **LEGv8** instructions for future reference, and we will cover these in the coming lectures.

![[armOverview.pdf]]