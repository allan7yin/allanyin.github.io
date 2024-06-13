### Weighted Undirected Graph

In the previous lectures, we have looked at shortest path problem given every edge had weight 1 → computed a predecessors array via **BFS** and then work backwards from target node (say $v$﻿) all the way back to the source node $s$﻿, adding every node along the way into a stack. Now, we look at instances where graph edges are not simply weight 1, but some arbitrary weight. In this course, we will only look MST on undirected graphs. We review some graph theory concepts:

> **MST:** a tree (connected acylic graph) that includes every node, and **minimizes** the total sum of edge **weights**
> 
> ![[Screenshot_2023-11-19_at_1.31.17_PM.png]]
> 
> The MST has branches highlighted in blue

What are some applications of **MST?**

- Was developed in the 50’s when lots of phone lines were being established
- Telecom companies wanted some method to connect all these nodes to their network to receive phone service, in the cheapest manner possible

**→ Useful Tree Facts:**

- A tree on $n$﻿ vertices has $n-1$﻿ edges
- There is a unique path between any 2 vertices in a tree
- If $T$﻿ is a tree and an edge $c \notin T$﻿ is added T, then the resulting graph contains a unique cycle $C$﻿
- if $e' \in C$﻿, then $T \cup \{e\} \backslash \{e'\}$﻿ is a tree

> **A cut of a Graph:** a **cut** in a graph $G = (V,E)$﻿ is a partition of $V$﻿ into 2 non-empty subsets $S$﻿ and $V \backslash S$﻿

> **A cut-set of a cut:** given a cut $(S, V \backslash S)$﻿, the **cutset** is the set of edges with one endpoint $S$﻿ and the other in $V \backslash S$﻿
> 
> ![[Screenshot_2023-11-19_at_3.15.45_PM.png]]
> 
> Yellow is one cut, white is another cut. The **blue** edges are the edges in the **cut-set**

**Theorem: Cut Property**

For any cut $(S, V \backslash S)$﻿ of a graph $G$﻿, the **minimum weight** edge in the **cut-set** is in every MST for $G$﻿. For example, the orange edge in the above diagram is the **minimum weight edge →** consequently, it is in every MST of $G$﻿. How to prove?

- Let $e = (u,v) $﻿ be the **lightest edge crossing (****$u \in S, v \in V \backslash S$**﻿**)**
- Let $T$﻿ be a MST and suppose that $e \notin T$﻿ for the sake of contradiction
- We construct spanning $T'$﻿ s.t. $w(T') < w(T)$﻿ for contradiction
- $T$﻿ is spanning, so there exists some path $u \rightsquigarrow v$﻿
- The path starts in $S$﻿ and ends in $V \backslash S$﻿, so it contains an edge $e' = (u', v'), u' \in S, v' \in V \backslash S$﻿
    - so, this edge crosses the cut
- Let $T' = T - \{e'\} + \{e\}$﻿
    - So, $T'$﻿ is still a spanning tree and in fact, since $e$﻿ is lightest edge crossing the cut → $w(T') < w(T)$﻿, which is a contradiction, as we assumed $T$﻿ was the **minimum spanning tree**. Thus, $T$﻿ cannot be a MST if it does not contain $e$﻿.

---

### Kruskal’s Algorithm

So, how do we build an MST? We use **Kruskal’s algorithm**. This is a greedy algorithm, and here how it works:

- Sort edges in increasing weight, from lightest to heaviest.
- For each edge in this sequence, we add it to $T$﻿ if upon adding it, no cycle is created

![[Screenshot_2023-11-19_at_4.56.35_PM.png]]

Here is what the execution of this algorithm would look like. How to prove this:

- Let $T$﻿ be a partial spanning tree just before adding $e = (u,v)$﻿, the lightest edge that does not create a cycle
- Let $S$﻿ be the connected component of $T$﻿ (a subgraph in which there is a path between every pair of vertices) that contains $u$﻿
- Note that $e = (u,v)$﻿ cross the cut $(u \in S, v \in V \backslash S)$﻿, or it would create a cycle
- Out of all the edges crossing the cut, $e$﻿ is considered first, so it is the **lightest** of these edges

  

So, how do we implement this? Sorting is straightforward. Howe do we determine if adding edge $e$﻿ would create a **cycle?** We take a look at the **Union Find Algorithm.** This is a data structure and algorithm used to efficiently keep track of a partition of a set into disjoint subsets. There are 2 main operations:

- `**Union(**``**$S_i, S_j)$**`﻿ → merges 2 sets into one set → replaces the 2 sets with $S_i \cup S_j$﻿
- `**Find**``**$(e_i)$**`﻿ → returns the **label** of the set containing $e_i$﻿

  

**→ Kruskal’s Using Union-find**

- Each node is initially its own subset
- Add an edge → union 2 subsets
- An edge **creates a cyle iff** its endpoints are in the same subset

![[Screenshot_2023-11-19_at_5.47.36_PM.png]]

What is the above showing? In the beginning, we have a,b,c,d as separate sets. So:

- We look at the lightest edge → 2 → are nodes a and b in separate set? Yes. So, union their sets and add edge a,b to $T$﻿
- We continue this until we only have on set: a,b,c,d → so, we cannot add the heaviest edge, as both d and c are in the same set, and would cause $T$﻿ to have a cyce

```Python
def Kruskal(V[1...n], E[1...m]):
	sort E[1...m] in increasing order by weight 
	uf = new UnionFind data structure 
	mst = new List

	for j = 1...m:
		set_a = uf.find(E[j].source)
		set_b = uf.find(E[j].target)
		if set_a != set_b:
			mst.add(E[j])
			uf.merge(set_a, set_b)

	return mst
```

What is the time complexity of this algorithm? Well, we would need to know the runtime or `UnionFind`, which is a complex topic. This likely **is not important** → but for the sake of saying, a simple implementation makes it operations $log(n)$﻿. So, the runtime of **kruskal’s** is $O(mlogn)$