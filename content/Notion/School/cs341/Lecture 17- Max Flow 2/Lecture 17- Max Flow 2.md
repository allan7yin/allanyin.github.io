To begin, we continue where we left off for **Ford-Fulkerson Method.**

---

→ Find a **shortest path** **$P$**﻿ from $s$﻿ to $t$﻿ in the **residual graph**

- If it improves the **flow,** we call it an **augmenting path**, and use it to update the **flow**
- In fact, any st-path in the residual graph is an **augmenting path**

![[Screenshot_2023-11-21_at_10.07.11_PM.png]]

This before → for each forward edge in residual graph, increase by this amount in graph

![[Screenshot_2023-11-21_at_10.10.43_PM.png]]

So, in left one, flow is increased on corresponding edges → **ONLY FORWARD EDGES**

![[Screenshot_2023-11-21_at_10.17.08_PM.png]]

For this one → backward edge in P → decrease by the amount in original edge

Now, based on the residual graph, we have modified our original flow. We now construct a new residual graph from this new graph. If no s-t path $P$﻿ can be found in the new residual graph → **Done!**

![[Screenshot_2023-11-21_at_10.20.49_PM.png]]

So, let’s construct an algorithm from this understanding.

- An **augmenting path** w.r.t a flow $f$﻿ is a **simple** s-t path in $R_f$﻿ (simple means no cycle)
- Let $P$﻿ be an **augmenting path** w.r.t to $f$﻿
- Let $bottleneck(f,P)$﻿ be the minimum capacity of an edge in $P$﻿
- We show that the following subroutine $augment(f,P)$﻿ always improves the value of flow $f$﻿

```Python
# this function makes the changes to the edges in P -> reflected in the original path
def augment(f,P):
	let b = bottleneck(f,P)
	for each edge e = (u,v) in P
		if e is a forward edge:
			f(e) = f(e) + b
		else if e is a backwards edge:
			let e' = (v,u)
			f(e') = f(e') - b
```

  

**→ Lemma 4:** `**Augment()**` **Improves Flow** **$f$**﻿

- Let $f$﻿ be a flow in $G$﻿ with $f^{in}(s) = 0$﻿ and $P$﻿ be an augmenting w.r.t $f$﻿
- Let $f'$﻿ be the resulting flow after running `augment(f,P)`
- Then $f'$﻿ **is a flow with** **$value(f') = value(f) + bottleneck(f,P)$**﻿
- That is `augment(f,P)` **increases the flow by** `**bottleneck(f,P)**`

  

**→ Proof**

- **Claim:** `**augment(f,P)**` **increases the flow by** `**bottleneck(f,P)**`
- First, check $f'$﻿ is a flow
    - Prove capacity and conservation constraints, and $f^{in}(s) = 0$﻿
- **Are capacity constraints satisfied?**
    - We add/subtract `bottleneck(f,P)` to/from each edge
    - And `bottleneck(f,P)` is the minimum of the smallest remaining capacity and the current flow
    - So, capacity constraints are satisfied
- **Claim:** `**augment(f,P)**` **increases the flow by** `bottleneck(f,P)`
- **How about conservation of flow?**
    - Consider how the flow into and out of each vertex $u \notin \{s,t\}$﻿ changes as a result of running `augment(f,P)`
        - We show the change in $f^{in}(u)$﻿ is the same as the change in $f^{out}(u)$﻿
        - There are 4 cases → depending on whether the edges entering/leaving $u$﻿ are **forward** or **backward** edges

![[Screenshot_2023-11-21_at_11.16.49_PM.png]]

![[Screenshot_2023-11-21_at_11.16.58_PM.png]]

**→ Showing** **$f'^{in}(s) = 0$**﻿

- We showed capacity constraints are satisfied. Now, the last step in showing **$f'$**﻿ is a flow → **prove** **$s$**﻿ **still has no flow into it**
- Since $f$﻿ is a flow → $f^{in}(s) = 0$﻿
- To get $f'^{in}(s) > 0$﻿, an augmenting path must include an edge **into** **$s$**﻿
- But then, an augmenting path starts at $s$﻿ and returns to $s$﻿ → a cycle → a **contradiction!**

  

**Finally,** we argue $value(f') = value(f) + bottleneck(f,P)$﻿

- $f$﻿ and $f'$﻿ are flows, so $value(f') = f'^{out}(s)$﻿ and $value(f) = f^{out}(s)$﻿

- We thus show $f'^{out}(s) = f^{out}(s) + bottleneck(f,P)$﻿
    - The augmenting path $P$﻿ is a **simple** path (leaving $s$﻿ exactly once)
    - There is no flow into $s$﻿, so the edge $e \in P$﻿ leaving $s$﻿ is a **forward edge.** This means `augment(f,P)` **adds** `bottleneck(f,P)` to $f(e)$﻿
- So, $f'^{out}(s) = f^{out}(s) + bottleneck(f,P)$﻿

  

So now, we have the pseudo-code:

```Python
def FordFulkerson(G=(V,E)):
	for e in E:
		f(e) = 0
	
	while there is a simple s-t path P in Rf do:
		augment(f,P)
		an update the residual graph Rf
```

---

### Max-Flow Min-Cut Theorem Proof

What have we proved so far? → **augmenting improves flow**. We don’t know yet if:

1. We can actually obtain the **max-flow**, or
2. Whether **max-flow = min-cut**

  

**→ Proof Strategy**

- **Claim:** when there is **no augmenting path,** there is a **cut with capacity equal to** the value of the **current flow**
- Proving this will simultaneously:
    - prove the max-flow min-cut theorem
    - prove correctness of the **Ford-Fulkerson** method
    - solve the max-flow problem, and
    - solve the min-cut problem

So, as usual, we prove this in 2 directions → **max flow ≤ min cut**, and **max flow ≥ min cut**. We have already proven the the first on in the previous lecture. So, we now look to prove that **max flow ≥ min cut:**

- To prove this, we make the following proposition: **If** **$f$**﻿ **is an st-flow such that there is no st-path in the residual graph** **$R_f$**﻿**, then there is an st-cut** **$S$**﻿**, such that** **$value(f) = c^{out}(S)$**﻿
    
    ![[Screenshot_2023-11-22_at_8.55.07_AM.png]]
    
- Since there is no st-path in $R_f$﻿, there is a subset of $S$﻿ of vertices with $s \in S, t \notin S$﻿ such that $S$﻿ has **no outgoing edges** in $R_f$﻿. The below illustrates this:
    
    ![[Screenshot_2023-11-22_at_9.00.08_AM.png]]
    
    - **Claim:** **$c^{out}(s) = value(f)$**﻿
    - Consider 2 types of edges:
        - **Type 1:**
            - $uv$﻿ **exiting** **$S$**﻿ **in** **$G$**﻿, so $uv \in \delta^{out}(S)$﻿ in $G$﻿, $u \in S, v \notin S$﻿ → $uv$﻿ is in the set of edges **directed out from** **$S$**﻿
            - Since $S$﻿ has no outgoing edge in $R_f$﻿, we know $uv \notin R_f$﻿
            - This implies $f(uv) = c(uv)$﻿ → flow of the edge = capacity of the edge
                - flow cannot be less than capacity, since this would result in 2 edges, one forward and one backwards, and we know there is no forward edge as there is no edge leaving $S$﻿ in $R_f$﻿
        - **Type 2:**
            - $uv$﻿ **entering** **$S$**﻿ **in** **$G$**﻿, so $uv \in \delta^{in}(S)$﻿ in $G$﻿, $u \notin S, v \in S$﻿→ $uv$﻿ is the set of edges **directed into** **$S$**﻿
            - Since $S$﻿ has no outgoing edges in $R_f$﻿, we know there is no edge $vu \notin R_f$﻿
            - This implies $f(uv) = 0$﻿ → otherwise, we would have a backwards edge going outwards in $R_f$﻿ with capacity $c(uv)-f(uv)$﻿. But since no such edge, must be 0
- So, we have just showed that:
    
    - For edge $uv$﻿ directed out of $S$﻿, $f(uv) = c(uv)$﻿
    - For edge $uv$﻿ directed into $S$﻿, $f(uv)= 0$﻿
    - So, $f^{out}(S) - f^{in}(S) = c^{out}(S) - 0 = c^{out}(S)$﻿
        - Why is this true? $f^{out}(S)$﻿ is the sum of all flows for out edges of the cut $S$﻿ → since every edge $uv$﻿ directed out of $S$﻿ has flow equal to capacity, in the above we have $f^{out}(S) = c^{out}(S)$﻿
        - Same reasoning applies for $f^{in}(S) = 0$﻿
    - This proves the proposition. I.e. **If** **$f$**﻿ **is an st-flow such that there is no st-path in the residual graph** **$R_f$**﻿**, then there is a cut matching the flow**
        - Why is this? Remember, $value(f)$﻿ is the same as the flow over the **minimum st-cut** → this proof above shows that there exists a minimum st-cut $S$﻿, and that its flow value is equal to $c^{out}(S)$﻿, which in turn means the value of the flow $f$﻿ is equal to $c^{out}(S)$﻿ → **MAYBE ASK FOR CLARIFICATION**
    
    > [!important]  
    > Clarify the above → in particular, connecting it to max flow ≥ min cut  
    

> **So from Ford-Fulkerson, how we can find the Min-Cut? → After algorithm has terminated, check all nodes reachable from source** **$s$**﻿**, these nodes make up** **$S$**﻿**,as we know from above, with no augmenting path, there will eventually be some** **$S$**﻿**, where there are no outgoing edges (so just the reachable nodes from** **$s$**﻿**).** Note, in the very top of example of this lecture, the set $S$﻿ is just the node $s$﻿, which is completely fine. Once we have set $S$﻿, we can go back to flow graph. The value of the graph is continually improved during **Ford-Fulkerson →** and from above, we know that the capacity of the out-going edges in $S$﻿ in $G$﻿ is = $value(f)$﻿.  
>   
> Note, Ford-Fulkerson does not explicitly return the value of the flow → we will need to use  
> $S$﻿ to find it based on the edge capacities  
>   
> → This was prompted from  
> [https://piazza.com/class/lm6cv4xzhai4t2/post/656](https://piazza.com/class/lm6cv4xzhai4t2/post/656)

---

### Time Complexity: Ford-Fulkerson Method

This depends on the implementation of the algorithm. As a refresher, this is the algorithm:

```Python
def FordFulkerson(G=(V,E)):
	for e in E:
		f(e) = 0
	
	while there is a simple s-t path P in Rf do:
		augment(f,P)
		an update the residual graph Rf
```

- How do we find an augmenting path?
- How many times do we need to augment before we terminate?

  

**→ Runtime**

- Assume we find any arbitrary augmenting path $P$﻿, using any technique (BFS or DFS) → $O(m+n)$﻿ time
- The, every time `augment(f,P)` is run, we know only that flow **increases**
- If capacities are **integers,** the increase is at last 1
- In this case, if **max flow is** **$k$**﻿**, then runtime is** **$O(k(n+m))$**﻿ **→ there are** **$k$**﻿ **paths and each path takes** $O(m+n)$﻿ time
    - For max flow, we assume a connected graph so this is $O(km)$﻿ → no need for the outer loop (recall BFS) → just iterate over edges
    - **VERY BAD IF** **$K$**﻿ **IS LARGE!**

![[Screenshot_2023-11-22_at_10.53.44_AM.png]]

![[Screenshot_2023-11-22_at_10.54.08_AM.png]]

→ Edmonds-Karp is a specific implementation of the **Ford-Fulkerson** algorithm. It uses BFS to find the augmenting path → **run time of Edmonds-Karp’s implementation of Ford-Fulkerson algorithm is** **$O(nm^2)$**