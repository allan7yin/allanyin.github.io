### All Pairs Shortest Paths (APSP) Problem

The All Pairs Shortest Paths (APSP) problem is a classic problem in computer science and graph theory. The goal of this problem is to find the shortest paths (smallest sum of weights) between all pairs of vertices in a given weighted graph. In other words, for every pair of vertices (u, v), the algorithm should find the shortest path from vertex u to vertex v.

- **Instance:** A digraph $G = (V,E)$﻿, and a **weight matrix** **$W$**﻿, where $W[i,j]$﻿ denotes the weight of edge $ij$﻿, for all $i,j \in V, i \ne j$﻿
- **Find:** For all pairs of vertices $u,v \in V, u \ne v$﻿, a directed path $P$﻿ from $u$﻿ to $v$﻿ such that:
    - $w(P) = \sum_{i,j \in P} W[i,j]$﻿ is minimized
    - Basically, the weight of the **path** is the sum of weights of the edges in the path → minimized
- We allow edges to have negative weights, but we assume there are no **negative-weighted** directed cycles in $G$﻿

  

![[Screenshot_2023-11-21_at_1.02.08_PM.png]]

  

We use the following conventions for the weight matrix $W$﻿:

$x = \begin{cases}$﻿

The easy solution would be to run **Bellman-Ford** n times → as **Bellman-Ford** finds the shortest path from source node $s$﻿ to every other node in the graph. This can potentially be **very** expensive.

![[Screenshot_2023-11-21_at_1.04.23_PM.png]]

**→ A Dynamic Programming Approach**

![[Screenshot_2023-11-21_at_1.41.45_PM.png]]

Suppose we successively consider paths of length $1,2,...,n-1$﻿. Let $L_m[i,j]$﻿ denote the minimum-weight (i,j)-path having at most $m$﻿ edges → We want to compute $L_{n-1}$﻿.

- The base case is $L_1 = W$﻿ → That is, just the weight matrix, as every single edge is a path of length 1.
- The recursive relation is: $\text{For } m \ge 2, L_m[i,j] = min\{L_{m-1}[i,k] + L_1[k,j]: 1 \le k \le n\}$﻿
    - That is, we want to try this for **all possible** predecessors $k$﻿
    - Only problem is, we don’t know the **predecessor** of $j$﻿ on the optimal path $P$﻿

← Here is what some pseudo-code may look like. Let’ go over it:

- Outer → We iterate over the possible lengths of paths: `2 → n-1` (length 1 path is base condition)
- For each length, we iterate over all pair of vertices in the graph → for all $i,j$﻿
    - For each vertex pair, we perform the recursive application → We iterate over all **potential predecessors** of $j$﻿, so we need to iterate over all nodes. This is fine as we are taking the `min` with $\infin$﻿, so if `k` is not a predecessor, then `W[k,j` is already $\infin$﻿, so we just set $l = \infin$﻿, as it will be overwritten in subsequent iterations (or perhaps it has already been set, in which case this has no effect on $l$﻿)
- This is a $O(n^4)$﻿ run-time
- We can improve this with a technique known as **successive doubling**

**→ Better Solution: Successive Doubling**

![[Screenshot_2023-11-21_at_2.34.05_PM.png]]

  

Successive doubling typically refers to a strategy where the size of a problem or a data structure is doubled in each step. We apply this principle to the above DP solution. The idea is to construct $L_1, L_2, L_4,..., L_{2^t}$﻿, where $t$﻿ is the smallest integer such that $2^t \ge n-1$﻿. So, we no longer need to construct $L_3, L_5, ...$﻿ as they are contained inside of the even $L$﻿ on top of it. For example, with this improvemenet, we do not need to compute $L_3$﻿ directly → $L_4$﻿ accounts for all paths of length ≤ 4.

- **Base Case:** The same as before $L_1 = W$﻿
- Below is a quick demonstration of the intuition
    
    ![[Screenshot_2023-11-21_at_2.44.00_PM.png]]
    

**⇒ Come back to this later on for clarification!**

So, in the above, we have seen 2 algorithms:

1. Sub-problem is a path to the **predecessor node** → try all possible predecessor nodes $k$﻿
2. Sub-problems are paths to/from the **midpoint node** → try all possible midpoint nodes $k$﻿

  

The third, and best, solutions is also a DP solution, in which:

- Sub-problems are paths in which **all interior nodes** are in $\{1...k-1\}$﻿
    - i.e we **restrict paths** to using a **prefix** of all nodes → try all ways to use new node $k$﻿ as an interior node

---

### 3rd Sol: Floyd-Warshall Algorithm

This is the third solution, and the one that is the best so far. So, what is the intuition behind it? Similar to the previous 2 solutions we have seen, we **Floyd-Warshall** can be implemented with a dynamic programming approach. The following video is a fantastic in explaining it, and will serve as an excellent refresher during exam review:

[https://www.youtube.com/watch?v=4NQ3HnhyNfQ](https://www.youtube.com/watch?v=4NQ3HnhyNfQ)

The main idea behind **Floyd-Warshall Algorithm** is to gradually **build up all intermediate routes between nodes** **$i$**﻿ **and** **$j$**﻿ to find the optimal path. The basic idea is illustrated below:

![[Screenshot_2023-11-21_at_3.41.26_PM.png]]

![[Screenshot_2023-11-21_at_3.41.54_PM.png]]

  

So, what will out **DP** table look like? Let $D_k[i,j]$﻿ denote the length of the minimum-weight **path** **$i \rightsquigarrow j$**﻿ in which all **interior nodes** are in the set $\{1,...,k\}$﻿. We want to compute $D_n$﻿ → in which case, all $n$﻿ nodes are potential **interior nodes** (of course, not every node is an interior path for other $i,j$﻿ paths, just **potential options**). So, we will have $D_i \forall i \in n$﻿, which is also sort of like just a singular 3D DP array → `dp[k][i][j]` = **shortest path from** **$i$**﻿ **to** **$j$**﻿ **routing through nodes** **$\{0,1,...,k-1,k\}$**﻿**.** We can make an optimization, as we know we will only ever look back to `dp[k-1]` → Just make two 2D DP-arrays, and swap them back and forth between “current” and “previous”. This will be shown in the pseudo-code later on. So, what is the recurrence relation?

- In the very beginning, we have **base case →** **$D_0 = W$**﻿ → Weight Matrix
- For each node in the set $\{1,...,k\}$﻿:
    - **case 1:** **$k$**﻿ **is not in** **$P$**﻿
        - $D_k[i,j] = D_{k-1}[i,j]$﻿
    - **case 2:** **$k$**﻿ **is used in** **$P$**﻿
        - $D_k[i,j] = D_{k-1}[i,k] + D_{k-1}[k, j]$﻿
    - We can combine the following into 1: $D_k[i,j] = min(D_{k-1}[i,j], D_{k-1}[i,k] + D_{k-1}[k, j])$﻿

![[Screenshot_2023-11-21_at_3.55.43_PM.png]]

```Python
def FloydWarshall(W[1...n, 1...n]):
	D0 = copy of weight matrix W
	D1 = new n*n matrix 
	Dlast = pointer to D0
	Dcurr = pointer to D1

	for k = 1...n:
		for i = 1...n:
			for j = 1...n:
				Dcurr[i,j] = min(Dlast[i,j], Dlast[i,k] + Dlast[k,j])
		temp = DLast
		Dlast = Dcurr
		Dcurr = temp

	return Dlast # got swapped with Dcurr in the last outer loop iteration
```

![[Screenshot_2023-11-21_at_4.16.29_PM.png]]

---

### Stable Matching Problem

Consider the following problem description:

> **Stable Matching  
> Instance:  
> **
> 
> - 2 sets of size $n$﻿ → $X = [x_1,..., x_n]$﻿ and $Y = [y_1, ..., y_n]$﻿. Each $x_i$﻿ has a **preference ranking** of the elements in $Y$﻿ and vic versa.
> - $pred(x_i, j) = y_k$﻿ if $y_k$﻿ is the $j$﻿-th favourite element of $Y$﻿ of $x_i$﻿; and $pref(y_i, j) = x_k$﻿ if $x_k$﻿ is the $j$﻿-th favourite element of $X$﻿ of $y_i$﻿
> 
> **Find:**
> 
> - A **matching** of the sets $X,Y$﻿, such that there does not exist a pair $(x_i, y_j)$﻿ which is not in the matching, but where $x_i$﻿ and $y_j$﻿ prefer each other to their existing matches → a matching with this property is a **stable matching**

So, what is this saying? Imagine we have $n$﻿ boys and $n$﻿ girls, and we are trying to marry each pair. Each girl/boy has their own preference list, as to who they would like to be with. Now, obviously, not everyone can get their first choice (well technically they can, but usually things don’t work out like that). We say a matching of boy to girl is **stable** if there is no pair of boy,girl that prefers each other more than their own match.

- Say we have $(b1,g1)$﻿ and $(b2,g2)$﻿ as 2 boy, girl matches in the matching
- If $b1$﻿ preferred $g2$﻿ over his match, $g1$﻿ and at the same time, $g2$﻿ prefers $b1$﻿ over her match → then they should be together instead → hence, **unstable**

  

**→ A stable matching always exists, we will look at an algorithm to find one**

### Gale-Shapley Algorithm

How does this algorithm work?

- Elements of $X$﻿ propose to elements of $Y$﻿
- If $y_i$﻿ accepts a proposal from $x_i$﻿, then the pair $\{x_i, y_i\}$﻿ is **matches**
- An unmatched $y_i$﻿ must accept a proposal from any $x_i$﻿
- If $\{x_i, y_j\}$﻿ is a matches pair, and $y_j$﻿ subsequently receives a proposal from $x_k$﻿, where $y_j$﻿ prefers $x_k$﻿ over $x_i$﻿, then $y_j$﻿ accepts and the pair $\{x_i, y_j\}$﻿ is broken, and replaced with the new pair $\{x_k, y_j\}$﻿
    - If $y_j$﻿ **does not prefer** **$x_k$**﻿ **over** **$x_i$**﻿**, then nothing happens, and the pair is maintained**
- A matched $y_j$﻿ never becomes unmatched → only ever shifted to a new $x$﻿
- An $x_i$﻿ might make a number of proposals → determined by $x_i$﻿’s preference order

```Python
def GaleShapley(X,Y,pref):
	Match = new set()
	while there exists an unmatched x_i
		let y_j be the next element in x_i's preference list 
		if y_j is not matched:
			Match.add([x_i, y_j])
		else:
			x_k, y_j = Match.find(y_j)
			if y_i prefers x_i to x_k
				Match.remove([x_k, y_j])
				Match.add([x_i, y_j])
				x_k is unmatched 
				x_i is matched 
	return Match
```

Some quick notes:

- May need additional functionality to the set → searching by one parameter → maybe a set and a hashmap? $y_j$﻿ goes in the set which is hashed to $x_i$﻿ in a separate hashmap
- Maintain a set of which $x_i$﻿ are matched and which ones are not

**→ Proof of Correctness**

First, we need to show this algorithm always terminates → it is impossible that an unmatched $x_i$﻿ has proposed to every $y_j$﻿

- Once an element of $Y$﻿ is matched, it is never unmatched
- If $x_i$﻿ has proposed to every $y_j$﻿, then every $y_j$﻿ is matched → implies every element of $X$﻿ is matched → **contradiction**

  

**→ Proof of Optimality (Matching is Stable)**

- Assume for the sake of contradiction, that there is instable match: → we have pairs $(x_i, y_j)$﻿ and $(x_k, y_l)$﻿ where $x_i$﻿ prefers $y_l$﻿ over $y_j$﻿ + $y_l$﻿ prefers $x_i$﻿ over $x_k$﻿. There are 3 cases to consider:
- $y_l$﻿ rejected $x_i$﻿’s proposal
    - This implies $y_l$﻿ already matched with someone more prefered than $x_i$﻿ → can only switch to a better partner, implies $x_k$﻿ is a better partner than $x_i$﻿ → **contradiction**
- $y_l$﻿ accepted $x_i$﻿’s proposal, but later accepted another proposal
    - Must be someone better, implies $x_k$﻿ is a better partner than $x_i$﻿ → **contradiction**
- $y_l$﻿ accepted $x_i$﻿’s proposal, and did not accept any subsequent proposal
    - Then $y_l$﻿ would be matched with $x_i$﻿, but it didn’t → **contradiction**

So, all 3 cases are impossible → so, there **cannot be any instability**

  

So, what is the runtime of the algorithm above? It really depends on the way we implement it:

![[Screenshot_2023-11-21_at_9.07.32_PM.png]]

  

---

### Formulating Graph Problems

If we can formulate a problem as a graph problem → chances are, there is an efficient non-trivial algorithm for solving the problem. Some problems will have a very natural graph formulation, while for others, we may need to choose a less intuitive graph formulation. We’ll take a look at 2 interesting problems:

  

**→ The RootBear Problem**

![[Screenshot_2023-11-21_at_9.26.04_PM.png]]

[https://piazza.com/class/lm6cv4xzhai4t2/post/597](https://piazza.com/class/lm6cv4xzhai4t2/post/597) → interesting read for this problem

  

**→ Reliable Networking Routing**

![[Screenshot_2023-11-21_at_9.28.05_PM.png]]

---

As this is the final **graphs** lecture, the following table is a nice summary:

![[Screenshot_2023-11-21_at_3.04.32_PM.png]]