In this lecture, we will be finishing up greedy algorithms and begin looking at **dynamic programming.**

## Interval Colouring

One example problem of implementing greedy algorithm is the interval colouring problem.

- **Instance:** A set $A = \{A_1, A_2, ..., A_n\}$﻿ of intervals.
- For $1 \le i \le n$﻿, $A_i = [s_i, f_i)$﻿ where $s_i$﻿ is the **start time** of interval $A_i$﻿ and $f_i$﻿ is the **finish time** of interval $A_i$﻿
- **Feasible Solution:** A c-colouring is a mapping $col : A $﻿ → $\{1,2...,c\}$﻿ that assigns each interval a colour such that 2 intervals receiving the same colour are always disjoint
- **Find:** A c-colouring of A with the minimum number of colours

![[Screenshot_2023-10-09_at_6.44.29_PM.png]]

We can solve this by pre-sorting the intervals in increasing order of start time.

![[Screenshot_2023-10-09_at_6.45.54_PM.png]]

![[Screenshot_2023-10-10_at_12.54.22_PM.png]]

Here is the pseudo-code for this:

```Python
def Preprocess(A[1...n]):
	sort A by increasing start time => Θ(nlogn)
	let s[1...n] be the start times in A
	let f[1...n] be the finish times in A
	return GreedyIntervalColuring(s,f)

def GreedyIntervalColouring(s[1...n], f[1...n]):
	d = 1 # d will hold the number of colours we have 
	colour[1] = 1 # interval 1 gets colour 1
	finish[1] = f[1] # finish[c] = finish time of last interval fo receive colour c

	for i in range(2, n): # for each interval in A_i => Θ(n)
		reused = False
			for c in range(1,d): => Θ(d)
				if finish[c] <= s[i] then:
					colour[i] = c
					finish[c] = f[i]
					reused = True # we reused a colour, will not add one then like below 
					break
			if not reused then: # if after everything, we did not reuse a colour, we need a new one 
				d += 1
				colour[i] = d
				finish[d] = f[i]
	return d
```

  

**⇒ Now**, we prove correctness of the algorithm

For this, we can either use a inductive proof, or provide a “slick” proof. We will show the “slick” proof:

- Let $D$﻿ denote the number of colours used by the algorithm and let $F_D$﻿ be the first inteval that has colour $D$﻿. We prove that $F_D$﻿ overlaps $D - 1$﻿ other **intervals at a single point in time.**
- This also tells us that for all intervals $L_i \forall i \in [1,2,...D-1]$﻿, these intervals overlap with interval $F_D$﻿. Since they all overlap, it means al of them end after interval $F_D$﻿ has started, meaning that $finish[L_i]$﻿ must be after $F_D$﻿ starts. Hence, we will need to use $D$﻿ colours.

  

The above pseudo-code indicates that we have a runtime of $O(nlogn + nd)$﻿. In this case, it might only be $O(nlogn)$﻿ if only a constant number of colours are needed. However, in worst case, $O(n^2)$﻿ if $n$﻿ colours are needed.

- Note, this algorithm has its inefficiencies, and we can look to make it more efficient.

  

**⇒ Improving this Algorithm**

The current algorithm is performing a linear search when the algorithm is comparing the start time of interval $A_i$﻿ with finish times of all previous colours. Instead, what we can do is to use a **minHeap** to keep track of the earliest available colour, so if that colour is not available, we know. every other colour is also not available. This will allow us a constant time method of determining if a colour is needed to be added. While the minHeap will have costs associated with moving the nodes within the tree, they are only $O(logn)$﻿ costs, which are fine in the context of this algorithm. Here are some visualizations of what we are doing:

![[Screenshot_2023-10-10_at_2.42.16_PM.png]]

![[Screenshot_2023-10-10_at_2.42.25_PM.png]]

So, we obtain the new and improved algorithm:

```Python
def Preprocess(A[1...n]):
	sort A by increasing start time => Θ(nlogn)
	let s[1...n] be the start times in A
	let f[1...n] be the finish times in A
	return GreedyIntervalColuring(s,f)

def GreedyIntervalColouring(s[1...n], f[1...n]):
	d = 1 # d will hold the number of colours we have 
	colour[1] = 1 # interval 1 gets colour 1
	h = new minHeap()
	h.insert([f[1], colour[1]]) # [finish time, colour]

	for i in range(2, n): # for each interval in A_i => Θ(n)
		(fc, c) = h.min() # the root of the min heap 
		if fc <= s[i] then:
			h.deleteMin()  => Θ(logn)
			colour[i] = c
		else:
			d += 1
			colour[i] = d
		h.insert([f[i], colour[i]])

	return d
```

---

## Dynamic Programming

Now, we start looking at DP (Dynamic Programming). What is DP? Dynamic Programming (DP) is a method used in computer science and mathematics to solve problems by breaking them down into simpler subproblems. It is particularly useful for optimization problems where the goal is to find the best solution among a set of possible solutions. DP is often used when a problem has overlapping subproblems and optimal substructure.

The key idea behind dynamic programming is to solve each subproblem only once and store the solution to avoid redundant computations. This is achieved through memoization or bottom-up approaches. The term "programming" in dynamic programming doesn't refer to coding; instead, it comes from the mathematical programming used in optimization.

$T(n) = T(n-1) + T(n-2) + O(1) \\$

  

We first introduce this through solving the problem of **computing Fibonacci numbers.** Well of course, we are familiar with the recursive approach. But, that approach is actually terribly inefficient. But why is the recursive approach so slow? let’s look close at it.

  

Consider the recursive way of generating fibonacci numbers:

```Python
def BadFib(n):
	if n == 0 or n == 1:
		return n
	return BadFib(n-1) + BadFib(n-2)
```

So for this, we can get the following analysis:

Here, we have $\frac{n}{2}$﻿ levels of recursion for the first expression, and $n$﻿ levels of recursion for the second expression, where work doubles at each level. So, we know that $T(n)$﻿ is **certainly** **$\Omega(2^{\frac{n}{2}})$**﻿ **and** **$O(2^n)$**﻿**.** But why is this so slow? It is slow because the subproblems we are recursing on, have overlaps, meaning we are repeating some amount of work over and over again. The following diagram demonstrates this perfectly:

![[Screenshot_2023-10-10_at_3.21.59_PM.png]]

So, this brings the question, how can we design dynamic programming algorithms for optimization problems? Here is sort of what the though process may look like.

  

**⇒ (Optimal) Recursive Structure**

Examine the structure of an optimal solution to a problem instance $I$﻿, and ==determine if an optimal solution for== ==$I$==﻿ ==can be expressed in terms of optimal solutions to certain== ==**subproblems**== ==of== ==$I. $==﻿

- **Define Subproblems:**
    - Define a set of subproblems $S(I)$﻿ of the instance $I$﻿, the solution of which enables the optimal solution of $I$﻿ to be computed. $I$﻿ will be the last or largest instance in the set $S(I)$﻿
- **Recurrence Relation:**
    - Express how the optimal solution to a given instance $I$﻿ of the problem can be recursively defined in terms of optimal solutions to smaller instances or base cases.
- **Compute Optimal Solutions**
    - Compute the optimal solutions to all the instances in $S(I)$﻿. Compute these solutions using the recurrence relation in a **bottom-up** fashion, filling in a table of values containing these optimal solutions.
    - Whenever a particular table entry is filled in using the recurrence relation, the optimal solutions of relevant subproblems can be looked up in the table (they have been computed already).
    - The final table entry is the solution to $I$﻿

  

With this structure in mind, we can go back and revisit **fibonacci problem**. We can define some sub-problems:

- The set of subproblems that will be combined to obtain $Fib(n)$﻿ is $\{ Fib(n-1), Fib(n-2)\}$﻿
- So, we know $S(I) = \{Fib(0), Fib(1),...Fib(n)\}$﻿

Also, we know the recurrence relation look something like

![[Screenshot_2023-10-10_at_8.40.31_PM.png]]

To compute the (Optimal) solutions, create **table f[1…n]** and compute its entries **“bottom-up”**

  

**⇒ Key Idea:**

- When computing a table entry, we must have **already computed** the **entries** it depends on! So, we need to compute the dependencies first. In the example of fibonacci, computing all fibonacci numbers from 1 to n solves this. Consider the following pseudo-code:

```Python
def FibDP(n):
	f = new array of size n

	f[0] = 0
	f[1] = 1

	for i = 2 ... n:
		f[i] = f[i-1] + f[i-2]
	
	return f[n]
```

Now, while there is a more space efficient solution, this suffices for our purposes.

**⇒ Correctness**

- **Step 1:** Prove that when computing a table entry dependent entries are **already computed** (which is the case by default for the above example)
- **Step 2:** Suppose subproblems are solved correctly (optimally). Prove these (optimal) sub-solutions are combined into a(n optimal) solution

When we analyze the cost of this algorithm, using the **unit cost model** is not very realistic for this problem, as Fibonacci numbers grow very quickly. **But, lets use it anyways for simplicity.** Under this cost model, we can conclude that the above pseudocode runs in $\Theta(n)$﻿

![[Screenshot_2023-10-10_at_11.48.46_PM.png]]

Why $log(n)$﻿? It is because the number of bits required to represent an integer grows logarithmically with the value of the integer.

For example, consider a binary representation of a number $x$﻿. The number of bits needed to represent $x$﻿ in binary is $\lceil log_2(x+1)\rceil$﻿. If you have a Fibonacci number $F(n)$﻿, the number of bits needed to represent it is proportional to $log_2(F(n)+1)$﻿.

  

## Rod Cutting

The previous example simply demonstrated what the principles of DP are, but didn’t really aim to optimize anything. We look at such an example now. This is the **rod cutting** problem.

- **Input:**
    - **n:** length of the rod
    - $p_1, p_2, ...p_n$﻿: $p_i$﻿ = price of a rod of length i
- **Output:**
    - Max **income** possible by cutting the rod of length **n** into any number of **integer** pieces (maybe **no** cuts)

![[Screenshot_2023-10-11_at_10.07.39_AM.png]]

![[Screenshot_2023-10-11_at_10.07.46_AM.png]]

To start, we want to think recursively. Given a rod of length **n,** we either make no cuts, or cut and recurse on the 2 remaining parts. The max income is the max income of the left part + the max income of the right part, or just the entire rod with no cut. We can branch this out to a more generalized format:

![[Screenshot_2023-10-11_at_10.53.39_AM.png]]

> **Note:** It is important in DP problems, to demonstate the existence of the optimal substructure, as this is the foundation for DP. Beginning with recurrence (top-down) is a good way to get a feeling for how we want to design the dp solution, which will be a bottom-up solution.

**⇒ Recurrence Relation:**

- Define $M(k) = $﻿ maximum income for a rod of length $k$﻿
    - **This is a critical step, we must define what the recurence relation means semantically**
- If we do not cut the rod, the max income is $p_k$﻿
- Say we cut the rod at $i$﻿:
    
    ![[Screenshot_2023-10-11_at_10.56.54_AM.png]]
    
- Max income is max of the left + max of the right: $M(i) + M(k-i)$﻿
- We are looking to maximize this income over all values of $1 \le i \lt k$﻿ → $max\{M(i) + M(k-i)\}$﻿
- So, for the entire rod of length k, we get: $max\{p_k, max_{1 \le i \le k-1}\{M(i) + M(k-i)\}\}$﻿

  

Now, we want to construct the bottom-up table, so we need to know what dependencies $M(k)$﻿ has, to be already present in the table.

![[Screenshot_2023-10-11_at_11.00.35_AM.png]]

This shows to compute $M(k)$﻿, we need to compute $M(k-1)$﻿. With this, we can form an algorithm. Consider the following pseudocode:

```Python
def RodCutting(n, p[1...n]):
	M = new array[1...n]

	# compute each entry M(k)
	for k = 1 ... n:
		M[k] = p[k] # current best = no cuts 
		
		for i = 1 ... (k-1):
			M[k] = max(M[k], M[i] + M[k-i])
	
	return M[n]
```

This has a time complexity of $\Theta(n^2)$﻿, but is this a quadratic time algorithm? In this problem, we likely won’t be dealing with gigantic numbers, and more with some reasonably sized nunbers. So, the **unit-cost** model is a apparopriate here. Each element of $p$﻿ is a word, so each element is stored in $\Theta(1)$﻿ bits. So, there are $\Theta(n)$﻿ words, with input size being $S\in \Theta(n)$﻿. So, $\Theta(n^2) = \Theta(S^2)$﻿, this indeed is a **quadratic** algorithm.

![[Screenshot_2023-10-11_at_11.07.34_AM.png]]