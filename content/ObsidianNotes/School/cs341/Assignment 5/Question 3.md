### Part 1:

For this problem, we will demonstrate that the $\text{Modified-St-Path} $﻿ problem is **NP-Complete** via a poly-time transformation from the known NP-Complete problem $\text{Directed-Hamiltonian-Path}$﻿. We prove this in 2 parts: **(1)** we show $\text{Modified-St-Path} $﻿ is NP and **(2)** we present a polynomial transformation.

**→ Showing** **$\text{Modified-St-Path}$**﻿ **is NP**

To show this is NP, we first define the data type of the certificates. For this problem, this will be an array of vertices, where each vertex can be represented with an integer $1...n$﻿. The desired **yes-certificate** format is an array of vertices that form a simple s-t path in the graph with total weight at most $k$﻿.

Now, we design a poly-time `verify(I,C)` algorithm, keeping in mind the oracle is non-deterministic:

```Python
def verify(I=(s,t,G,k,w), C = path):
	# check the first and last vertices are correct
	if path[0] != s or path[-1] != t: return false
	
	# check that the edges in path are valid in the graph
	for all consecutibe pairs (vi,vj) in path:                                                 # where path is [v1, v2, ...]
		if (vi, vj) not in E:
			return False                                                                           # oracle has returned an edge not in the graph

	# check for duplicate vertices in path -> need simple path
	vertex_set = set()
	for v in path:
		add v to vertex_set
	
	if |vertex_set| != |path| return False                                                       # this means there were duplicate nodes 0 -> not simple

	# calculate path weight and check if less than k
	sum = 0
	for all adjacent pairs e = (v1,v2) in path:
		sum += w[e]

	return sum <= k
```

→ We show the runtime for this algorithm is polynomial on the input size.

- The cost of this algorithm comes from iterating through the variables in the path. We know $|path| = n$﻿ as we can have at most n vertices (all vertices) in the s-t path. So, this loop is bounded above by $O(m)$﻿ which is then bounded by $O(n^2)$﻿.
- We analyze this with respect to the input size. Since we are in the unit-cost model, $s,t,k$﻿ are constant space. The weight array $w$﻿ has $m$﻿ entries in it. For this graph, we can use the input size cheat sheet, defining the input size of $G$﻿ to be $|V| = n$﻿.
- Thus, it can then be seen that the run-time is poly-time on the input size: runtime → $O(n^2)$﻿ = $O(Size(I)^2)$﻿

Now that we have the poly-time verification algorithm, we provide a correctness proof for it.

**→ Case 1: Let** **$I$**﻿ **be any yes-instance → find** **$C$**﻿ **such that** `**verify(I,C) = True**`

- Let $I$﻿ be any yes-instance of the $\text{Modified-St-Path}$﻿ problem. Since $I$﻿ is a yes-instance, that means there must exist a _simple_ s-t path $C$﻿, such that the path weight is $\le k$﻿. So, this will go through each check of the `verify(I,C)` algorithm, and we will see that:
    - The first vertex is $s$﻿ and the last one is $t$﻿ → will not return false in the first line
    - All the edges in $C$﻿ are indeed in the original graph $G$﻿, so the first `for` loop will not return false
    - There are no duplicate vertices since it is simple → after the second `for` loop will not return false
    - By definition, $C$﻿ has path weight $\le k$﻿, so the return statement, the verification algorithm will return true

**→ Case 2: If** `**verify(I,C)**` **is true → prove** **$I$**﻿ **is a yes-instance**

- Suppose `verify(I,C)` is true. Then this means we:
    - At the first line, we did not return false, this means this path does start at $s$﻿ and end at $t$﻿
    - At the first for loop, did not return false. This means all edges found in path $C$﻿ were in the graph $G$﻿, no extra or non-existent edges
    - After the second for loop, when we compared the size of the path to the path inserted into a set (which only allows for distinct elements), we did not return false, which means the path $C$﻿ has no duplicate vertices → simple
    - At the end, we returned true, this means the sum of the edge weights in $C$﻿ is $\le k$﻿
    - So $C$﻿ proves the existence of a simple s-t path with path weight $\le k$﻿

We have proven that $\text{Modified-St-Path}$﻿ is NP. Now, to prove that is is NP-Complete, we provide a polynomial transformation from the known NP-Complete problem: $\text{Directed-Hamiltonian-Path}$﻿. To do so, we first define a poly-time transformation function. Consider the following transformation:

- Consider any input $I$﻿ for the $\text{Directed-Hamiltonian-Path}$﻿ problem. So, the input is simply a directed graph $G=(V,E)$﻿.
- For each $e \in E$﻿, we assign the edge a weight of $-1$﻿, so that $\forall e\in E, w(e) = -1$﻿
- We assign the value $k = -(n-1)$﻿
- Now, we need to assign values for $s$﻿ and $t$﻿ to map to a $\text{Modified-St-Path}$﻿ problem. To do so, can perform the following step:
    - We add 2 new nodes $s,t$﻿ and add edges between all nodes and $s,t$﻿, with no edge directly between $s$﻿ and $t$﻿. We give each of these edges an edge weight of 0
    - We make $s$﻿ our source node and $t$﻿ our destination node
- We argue this transformation has polynomial run-time on the input size:
    - In total, we re-assign all edge weights → $O(m) \in O(n^2)$﻿
    - We add new edges between $s,t$﻿ to all other edges → $O(n)$﻿
    - So total runtime comes to → $O(n^2)$﻿
    - For this problem, we say the input size is $O(n)$﻿
    - **Thus**, it is clear that the run time is polynomial on the input size.
- This polynomial transformation applies a transformation to the input, and **a single call** to the oracle without modification

**→ We argue this transformation satisfies 2 key conditions:**

- → If **$I \in \mathcal{J}_{yes}(\text{Directed-Hamiltonian-Path})$**﻿ then $f(I) \in \mathcal{J}_{yes}(\text{Modified-St-Path})$﻿
    - Suppose $I$﻿ is a yes-instance for the DHP problem. That its, we’re given a directed graph $G$﻿ that contains a hamiltonian path. This hamiltonian path has a start node and destination node. By the transformation, this hamiltonian path now has a new head $s$﻿ and new tail $t$﻿. Now, say the hamiltonian path is $v_1 \rightarrow v_2\rightarrow ... \rightarrow v_n$﻿ with path weight $-(n-1)$﻿ since there are $n-1$﻿ edges and each edge has weight $-1$﻿. Now with the transformation, we have the s-t path $s \rightarrow v_1 \rightarrow v_2\rightarrow ... \rightarrow v_n \rightarrow t$﻿ which still has path weight $-(n-1)$﻿. By definition, it is a simple path as we only visit each node once, so we need to show this path has weight $\le k$﻿. In the transformation, we have defined $k$﻿ to be $-(n-1)$﻿. Therefore, this path is a simple s-t path that has path weight $\le k$﻿, and thus, $f(I) \in \mathcal{J}_{yes}(\text{Modified-St-Path})$﻿.
- ← If $f(I) \in \mathcal{J}_{yes}(\text{Modified-St-Path})$﻿ then $I \in \mathcal{J}_{yes}(\text{Directed-Hamiltonian-Path})$﻿
    - Suppose $f(I)$﻿ is a yes-instance of the modified s-t path problem. So, we know there is a simple s-t path that has path weight $\le k$﻿. Now, we know from the transformation, that each edge was weight $-1$﻿ and $k = -(n-1)$﻿. Since there is an s-t path with path weight $\le -(n-1)$﻿ of the form: $s \rightarrow v_1 \rightarrow v_2\rightarrow ... \rightarrow v_n \rightarrow t$﻿, then since $s \rightarrow v_1$﻿ and $v_n \rightarrow t$﻿ have edge weight 0, then that means the path $v_1 \rightarrow v_2\rightarrow ... \rightarrow v_n$﻿ must also have path weight $\le -(n-1)$﻿. Given that each edge was weight $-1$﻿, this tells us there must be $n-1$﻿ edges in $v_1 \rightarrow v_2\rightarrow ... \rightarrow v_n$﻿. So, consider removing nodes $s$﻿ and $t$﻿ and all associated edges. Now, the remaining graphs has $n$﻿ nodes. We know the existence of the path $v_1 \rightarrow v_2\rightarrow ... \rightarrow v_n$﻿ which has $n-1$﻿ edges and thus visits all vertices in the graph. So, by definition, this is a directed hamiltonian path → we obtain instance $I$﻿ such that $I \in \mathcal{J}_{yes}(\text{Directed-Hamiltonian-Path})$﻿.

Therefore, as we proved that $\text{Modified-St-Path}$﻿ is in NP and that there is a valid polynomial transformation from a NP-Complete problem, we conclude that $\text{Modified-St-Path}$﻿ is also **NP-Complete**.

---

### Part 2:

For this problem, we will demonstrate that the $\text{Subset-Sum-One} $﻿ problem is **NP-Complete** via a poly-time transformation from the known NP-Complete problem $\text{Subset-Sum-Zero}$﻿. We prove this in 2 parts: **(1)** we show $\text{Subset-Sum-One} $﻿ is NP and **(2)** we present a polynomial transformation.

**→ Showing** **$\text{Subset-Sum-One} $**﻿ **is NP**

To show this is NP, we first define the data type of the certificates. For this problem, this will be an set/array of integers. The desired **yes-certificate** format is an array of integers, such that there exists a subset $C$﻿ that has sum 1. Now, we design a poly-time `verify(I,C)` algorithm, keeping in mind the oracle is non-deterministic:

```Python
def verify(I, C):
	# I the array of integers, while C is the subset of I sums to 1
	if C not subset of I then return false
	return (sum(C) == 1)
```

We show this poly-time on the input size: Checking whether $C$﻿ is $I$﻿ can be done in polynomial time → for example, we could perform a binary search for each element in $C$﻿. There are $n$﻿ items in $I$﻿ and at most $n$﻿ items in $C$﻿, meaning this is at worst $O(nlogn)$﻿. Evidently, this certificate verification algorithm runs in poly-time with respect to the input size.

Now that we have the poly-time verification algorithm, we provide a correctness proof for it.

**→ Case 1: Let** **$I$**﻿ **be any yes-instance → find** **$C$**﻿ **such that** `**verify(I,C) = True**`

Given $I$﻿ is a yes-instance, that means there is some subset $C$﻿ that has sum equal to 1. For any such subset $C$﻿, `verify(I,C)` will return true.

**→ Case 2: If** `**verify(I,C)**` **is true → prove** **$I$**﻿ **is a yes-instance**

Say the verification algorithm returns true, this means that $C$﻿ is a subset, where its sum is 1. Therefore, $I$﻿ is a yes-instance for the problem.

We have proven that $\text{Subset-Sum-One} $﻿ is NP. Now, to prove that is is NP-Complete, we provide a polynomial transformation from the known NP-Complete problem: $\text{Subset-Sum-Zero}$﻿. To do so, we first define a poly-time transformation function. Consider the following transformation:

- For every integer in the input, we multiply it by 10
- Then, for each positive value $i$﻿, we add $i+1$﻿ to the set of integers as well
- Then, we can call the oracle for $\text{Subset-Sum-Zero}$﻿

Clearly, this transformation is polynomial on the input size.

**→ We argue this transformation satisfies 2 key conditions:**

- → If **$I \in \mathcal{J}_{yes}(\text{Subset-Sum-Zero})$**﻿ then $f(I) \in \mathcal{J}_{yes}(\text{Subset-Sum-One} )$﻿
    - Suppose $I$﻿ is a yes-instance for $\text{Subset-Sum-Zero}$﻿ problem. So, there exists a subset $C$﻿ that has sum equal to 0. Let’s consider the effects of the transformation on $C$﻿. Say $C = \{-x_i ,-x_{i+1},...-x_{j-1} + x_j + ... + x_k\}$﻿ with sum 0, where we’ve moved negatives to the left and positives to the right for easy of demonstration. So, after the transformation, this set will look like $C = \{-10x_i ,-10x_{i+1},...-10x_{j-1} + 10x_j + ... + 10x_k\} \cup \{ (10x_j+1) + ... + (10x_k + 1)\}$﻿. Notice that first part of the union still has sum 0, as scaling by a factor of 10 does not have effect on the sum. Now, notice that by replacing any of the positive terms in the left set, with that term itself + 1 found in the right set, then there exists a subset $S \sube C$﻿, where the sum of $S$﻿ is 1. Hence, stepping back, we then have a subset $S \sube C \sube I$﻿ that has sum equal to 1. Hence, $f(I) \in \mathcal{J}_{yes}(\text{Subset-Sum-One} )$﻿.
- If $f(I) \in \mathcal{J}_{yes}(\text{Subset-Sum-One} )$﻿ then **$I \in \mathcal{J}_{yes}(\text{Subset-Sum-Zero})$**﻿
    - Suppose $f(I)$﻿ is a yes-instance. So, this means there exists a subset with sum 1. That is, there is some set $\{x_i,...,x_k\}$﻿ where the sum is 1. By the construction of transformation, one of these numbers ends with 1 (every term in $f(I)$﻿ either ends in 0 or 1, and is either a multiple of 10, or one more than a multiple of 0) and the rest will end in 0. Those ending in 0 are negative and those ending in 1 are positive. During the transformation, for all 1-ending terms $x_i$﻿, the value $x_i - 1$﻿ is also in $f(I)$﻿. So, since for the one element ending one in the subset of $f(I)$﻿ that sums to 1, we replace it with it’s $x_i-1$﻿ counterpart. Now, we effectively decremented that subset’s sum by 1, thus making it 0. Now, we can remove all excess values ending in 1, and scale back down by dividing by 10. Regardless of this step, we have obtained a set of integers $I$﻿, where there is a subset with sum 0. Thus, $f(I) \in \mathcal{J}_{yes}(\text{Subset-Sum-One} )$﻿

Therefore, as we proved that $\text{Subset-Sum-One}$﻿ is in NP and that there is a valid polynomial transformation from a NP-Complete problem, we conclude that $\text{Subset-Sum-One}$﻿ is also **NP-Complete**.