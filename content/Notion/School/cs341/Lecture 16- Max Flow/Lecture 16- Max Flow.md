### Flows and Paths

Now, we will start considering **edge-disjoint** problems. Consider:

- **Input:** directed graph $G =(V, E)$﻿ and 2 vertices $s,t \in V$﻿
- **Output:** a maximal number of edge-disjoint paths in $G$﻿
- Paths $P_1$﻿ and $P_2$﻿ are **edge-disjoint** if they do not share any edges

  

**→ s-t Flows**

Let $G = (V,E)$﻿ be a directed graph where each edge $e \in E$﻿ has a **capacity** **$c(e) > 0$**﻿. An **s-t flow** assigns a number $f(e)$﻿ to each edge satisfying:

- **Capacity constraints:** $0 \le f(e) \le c(e)$﻿ for each edge
- **Conservation flow:** **$f^{in}(v) = f^{out}(v)$**﻿ **for** **$v \notin \{s,t\}$**﻿
- **Source** **$f^{in}(s) = 0$**﻿
- **Sink** **$f^{out}(t) = 0$**﻿

Here is a quick visual example:

![[Screenshot_2023-11-20_at_2.06.53_PM.png]]

![[Screenshot_2023-11-20_at_2.09.00_PM.png]]

Each edge is usually written in a $a/b$﻿ notation → **a** represents the current flow $f(e)$﻿, while **b** represents capacity $c(e)$﻿. We also now define what $f^{in}(u)$﻿ and $f^{out}(u)$﻿ are. For any given node $u$﻿, we define:

- $f^{in}(u) = \sum_{e \; into \; u} f(e)$﻿
- $f^{out}(u) = \sum_{e \; outof \; u} f(e)$﻿

Basically, the sum of flows going **into** and **out of** a node $u$﻿.

---

### Max s-t flow

- **Input:** directed graph $G=(V,E)$﻿ with capacities $c(e)$﻿ for each $e \in E$﻿, and 2 vertices $s,t$﻿
- **Output:** a flow from $s$﻿ to $t$﻿ with maximum value → i.e. that maximizes $f^{out}(s)$﻿
- **Motivation:**
    - Maybe we’re working with a network of pipes, and we’re interested in the maximum amount of liquid that can be pumped through

  

> **Connection between flows and paths  
> →  
> **A flow can always be decomposed into “capacity-disjoint” paths

> **→ Lemma 1: Decomposition of s-t Flow into capacity-disjoint s-t paths**
> 
> - Let $f$﻿ be an st-flow where $f(e)$﻿ is an integer for each $e \in E$﻿, $f^{in}(s) = 0$﻿ and $value(f) = k$﻿, where value is the total amount transferred from the **source** to the **sink.**
> - Then there are s-t paths $P_1, P_2, ...P_k$﻿ such that each **edge** **$e$**﻿ **appears in** **$f(e)$**﻿ **of these paths → each path as flow 1**
>     - For example, if we have an edge $f(e) = 3$﻿, then we know this appears in 3 paths. Below provides a small example
>         
>         ![[Screenshot_2023-11-20_at_7.25.19_PM.png]]
>         

  

- **Proof sketch by induction**
    - **Base:** when $k = 1$﻿, there is only one path
    - **Inductive Step:** Suppose the lemma holds for $k-1$﻿, show it holds for $k$﻿ (where $k \ge 2)$﻿
        - Consider the edges with **non-zero flow.** There must exist some **s-t path** **$P$**﻿ in these edges.
        - **Decrease** the flow for each edge in $P$﻿ by 1

The 2 diagrams below help to illustrate this:

![[Screenshot_2023-11-20_at_8.04.10_PM.png]]

![[Screenshot_2023-11-20_at_8.04.21_PM.png]]

So, there are **s-t paths** **$P_1, P_2, .., P_k-1$**﻿ such that each edge $e$﻿ appears in $f(e)$﻿ of these paths. By adding $P$﻿, we can obtain $k$﻿ such paths. So, when would this lemma be applied? The following is an example:

- Given a flow of value $k$﻿, where $f(e) \in \{0,1\}$﻿ for all $e \in E$﻿
- The lemme says the flow $f$﻿ can be decomposed into $k$﻿ edge-disjoint paths (since the maximum edge flow is 1, meaning the edge appears in one path)
- So, if our goal is to find $k$﻿ edge-disjoint paths, we can just focus in finding such a flow instead.
    - So, we don’t need to worry about which edges belong to which paths during the algorithm
- Can extract paths from such a flow by repeatedly doing:
    - BFS on the non-zero flow edges, identifying an s-t path, and decrementing the flows along the path

  

This may be confusing → Lets take a look at the following exercise:

![[Screenshot_2023-11-20_at_8.26.45_PM.png]]

![[JPEG_image-4001-9EE0-64-0.jpeg]]

---

## Flows and Cuts

**→ Upper Bounding the Max Flow for** **$G$**﻿

What is a good upper bound on the value of a flow? Also, how do we know a flow is maximal? The trivial upper bound is **sum of capacities of all edges.**

![[Screenshot_2023-11-20_at_10.14.19_PM.png]]

So, the sum of all capacities is 17, but the max-flow is 3. This is not a very tight, and not very useful, upper bound. But a slightly better analysis is:

- $min(c^{out}(s), c^{in}(t))$﻿
    - where $c^{out}(s) = \sum_{e \;out \;of \;s} c(e)$﻿ and $c^{in}(s) = \sum_{e \;into \;s} c(e)$﻿
- Tightly bounds max flow in this case. But does it work always? Consider the below:

![[Screenshot_2023-11-20_at_10.18.27_PM.png]]

So, what we use the above → $min(c^{out}(s), c^{in}(t)) = 30$﻿. But, the max flow in the above is 1. What now? Before moving forward, we present some new definitions:

**→ Definition:** An st-cut is a partition $(S, V\backslash S)$﻿ where $s \in S$﻿ and $t \in V\backslash S$﻿

![[Screenshot_2023-11-20_at_10.23.03_PM.png]]

For the edge-disjoint paths problem from above, where $c(e) = 1$﻿ for all $e$﻿, cut capacity is just the **number of edges crossing the cut**

- If an s-t cut has at most $k$﻿ edges crossing the cut, then there are at most $k$﻿ edge-disjoint s-t paths, since each s-t path has an edge crossing the cut

> **Lemma 2:** if an s-t cut $S$﻿ has a capacity $k$﻿, the value of every flow must be ≤ $k$﻿.

**Proof Sketch:**

- Assume for the sake of contradiction, that a flow with value $k' > k$﻿ exists
- By lemma 1, a flow with value $k'$﻿ can be decomposed into $k'$﻿ capacity-disjoint paths, each with **flow = 1**
- Each such path crosses the cut → consumes one unit of the cut’s capacity (up to $k'$﻿ total)
- But, the cut’s capacity is $k$﻿, where $k < k'$﻿, so the paths are not capacity-disjoint → Contradiction!

  

So, from the above, we know that if a s-t cut $S$﻿ has capacity $k$﻿, the value of every flow must be $\le k$﻿. This holds **for any st-cut, including the s-t cut** **$S$**﻿ **with minimum capacity.** So → **max st-flow ≤ min capacity over all possible s-t cuts.** in fact, it turns out max flow is **exactly** the min cut capacity → so we can solve the max flow problem by finding the min cut

---

### Min s-t cut

- **Input:** digraph $G= (V,E)$﻿ with capacities $\gt 0$﻿ for $e \in E$﻿, and 2 vertices $s,t$﻿
- **Output:** an s-t cut $S$﻿ with minimal capacity $c^{out}(S)$﻿

**→ Max-flow Min-cut theorem (Theorem 3):**

- Every max s-t flow has value equal to the capacity of a min s-t cut
- This is one of the most beautiful and important results in combinatorial optimization and graph theory

---

### Ford-Fulkerson Method

Before we dive into this method, let’s consider a naive algorithm attempt. For simplicity, try the edge-disjoint path problem first (unit capacities).

- **Greedy idea:** find the shortest s-t path (to use few edges) → then repeat on remaining edges

Here is an example of what we may find:

![[Screenshot_2023-11-21_at_11.56.07_AM.png]]

Now, it is apparent that the greedy solution is not the optimal solution. With such a greedy solution, it is difficult to decide on a path **permanentlY → Unclear** on how to find a path that **belongs** in the optimal solution.

  

The Ford-Fulkerson algorithm is a method for finding the maximum flow in a flow network. It is a more general “local search” algorithm which can **undo** previous decisions to improve the flow. We can improve the greedy flow by “pushing back” some flow using an **augmenting path through a residual graph (we will define these later).** Essentially, the idea of pushing back is instead of directing all flow through on edge, we reduce the amount of flow we direct through that edge, hence “pushing back” on that edge from the greedy solution.

![[Screenshot_2023-11-21_at_11.59.33_AM.png]]

So, now we introduce the concept of **residual graphs** and **augmenting paths:**

**→ Residual Graphs**

- A **residual graph** **$R_f$**﻿ is defined for a **given flow** **$f$**﻿, and a **graph** **$G$**﻿
- $R_f$﻿ has the same vertices as $G$﻿
- For each edge $e = uv$﻿ in $G$﻿
    - If $f(e) < c(e)$﻿, then $R_f$﻿ contains a **forward** edge $(u,v)$﻿ with the **remaining capacity** **$c(e) - f(e)$**﻿
    - If $f(e) > 0$﻿, then $R_f$﻿ contains a **backwards** edge $(v,u)$﻿ with **capacity** **$f(e)$**﻿ representing flow that could be “pushed back”

![[Screenshot_2023-11-21_at_12.04.13_PM.png]]

![[Screenshot_2023-11-21_at_12.04.30_PM.png]]

_**→ Notice that even in the**_ _**$R_f$**_﻿_**, there is a path from**_ _**$s$**_﻿ _**to**_ _**$t$**_﻿_**, this will be useful later on**_