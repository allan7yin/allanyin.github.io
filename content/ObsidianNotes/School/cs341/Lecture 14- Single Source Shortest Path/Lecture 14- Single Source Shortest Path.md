### Dijkstra’s Algorithm: Single-source shortest path in a graph with non-negative edge weights

So, we are looking to solve the single source shortest path problem, where the input isa graph $G = (V,E)$﻿ and a **non-negative** weight function $w(e)$﻿ defined for every edge $e$﻿. The problem we are looking to solve is:

- for every node $v \ne s$﻿, output a path $s \rightsquigarrow v$﻿ with the **smallest total weight (among all paths** **$s \rightsquigarrow v$**﻿**)**

For this problem, we will study a **directed graph**, but this can also be defined for undirected graphs.

  

**→ Illustrative example**

![[Screenshot_2023-11-19_at_8.42.09_PM.png]]

So, what is going on in here? Lets analyze this.

- First of all, we are starting from node $a$﻿. So, by default, it is optimal. We mark optimal distances with black circles (they are all white in the beginning)
- We also make the default distance of each node $\infin$﻿, easier when taking the `min` of things
- Then, we look at all the out edges, we “relax” the neighbours (or edges to neighbours) by updating the distance to those neighbours (if the distance of current node + edge weight is less than next node’s distance value).
    - So in the above, we would update node $d$﻿ with value 2 and node $c$﻿ with value 7.
- **→ From an optimal node, the lightest edge going out of it (we will prove this later)**
- So, we mark $d$﻿ as optimal
- Then, from $d$﻿ we go on to update $b$﻿
- We repeat this like this

```Python
def Dijkstra(adj[1...n], s):
	pred[1...n] = [null, null, ..., null]
	dist[1...n] = [infty, infty, ..., infty]
	pq = new priority queue
	
	dist[s] = 0
	for u = 1...n:
		pq.enqueue(u, dist[u]) # [node, priority]

	while pq is not empty:
		u = pq.dequeueMin()
		for v in adj[u]:
			if dist[u] + w(u,v) < dist[v]:
				dist[v] = dist[u] + w(u,v)
				pred[v] = u
				pq.changePriority(v, dist[v])
	
	return pred, dist
```

- So, let’s analyze the algorithm to the left
- The method for this is very similar to **BFS**
- We instantiate our `pred` and `dist` arrays like before
- Instead of using a queue, we now use a priority queue
- We `enqueue` all the nodes (except the start node) into the priority queue with distance $\infin$﻿
- While the priority queue is not empty → we `dequeue` the minimum priority node from it
- Inside the `for` loop, we are performing the neighbour relaxation
- So, what is `pq.changePriority(v, dist[v])`?
    - We want to change the priority for node `v` now
        
        - We go into the `pq` to change this
        
          
        

**→ Correctness: Intuition**

Dijkstra’s algorithm iteratively constructs a set → **OPT** → optimal set of nodes for which we **know** the shortest path from $s$﻿ (initially OPT = $\{ s \}$﻿). Every time we add a node to OPT, such as the start node, we relax its outoing edges. After we’ve done all the relaxation, we look at everything reachable from the nodes in OPT, pick the one that has the smallest distance, and that one gets added to OPT, and we repeat.

![[Screenshot_2023-11-19_at_10.24.50_PM.png]]

This helps show the intuition from above

To prove Dijkstra’s algorithm, we are looking to prove the statement:

**→ Theorem:** At the end of the algorithm, for all $u$﻿, $dist[u]$﻿ is exactly the total weight of the shortest $s \rightsquigarrow u$﻿ path

We prove this in 2 parts:

- $dist[u] \le $﻿ the total weight of the shortest $s \rightsquigarrow u$﻿ path (case ≤)
- $dist[u] \ge $﻿ the total weight of the shortest $s \rightsquigarrow u$﻿ path (case ≥)

  

**Proof:**

**→ case ≤**

- Let $P$﻿ be **any (not necssarily the shortest) arbitrary path** **$v_0 \rightarrow v_1 \rightarrow v_2 \rightarrow ... \rightarrow v_l$**﻿ where $v_0 = s$﻿ and $v_l = u$﻿
- For any index $j$﻿, let $L_j$﻿ denote $w(v_0 \rightarrow v_1 \rightarrow v_2 \rightarrow ... \rightarrow v_j)$﻿
    - Length/weight of the $j$﻿-th prefix of the path $P$﻿
- We prove by induction: $dist[v_j] \le L_j$﻿ for all $j$﻿

  

![[Screenshot_2023-11-20_at_8.37.37_AM.png]]

Why are we looking to prove $dist[v_j] \le L_j$﻿? $L_j$﻿ is the weight of the arbitrary path $P$﻿ up to the $j$﻿-th node. We’re actually showing then, since $P$﻿ is arbitrary, tha $dist[v_j]$﻿ is shorter than **every possible path**, from $s$﻿ to $v_j$﻿. So, then by induction, we will prove this for node $u$﻿, hence arriving at out desired conclusion.

- **base:** **$dist[v_0] = dist[s] = 0 = L_0$**﻿
- **inductive step:** suppose $\forall_{j>0}: dist[v_{j-1}] \le L_{j-1}$﻿
    - When `dequeueMin()` returns $v_{j-1}$﻿: we check if $dist[v_{j-1}] + w(v_{j-1}, v_j) \lt dist[v_j]$﻿
    - If so (shorter path to node $v_j$﻿ found), we set $dist[v_j] = dist[v_{j-1}] + w(v_{j-1}, v_j) $﻿
    - If not, $dist[v_j] \le dist[v_{j-1}] + w(v_{j-1}, v_j) $﻿
    - In both cases, $dist[v_j] \le dist[v_{j-1}] + w(v_{j-1}, v_j) $﻿
        - By IH → $dist[v_{j-1}] \le L_{j-1}$﻿
        - So $dist[v_j] \le L_{j-1} + w(v_{j-1}, v_j) $﻿
        - $L_{j-1} + w(v_{j-1}, v_j) = L_j$﻿ by definition
        - → so, $dist[v_j] \le L_j$﻿

  

**→ case ≥**

- Let $P'$﻿ be the path $s \rightarrow ... \rightarrow pred[pred[u]] \rightarrow pred[u] \rightarrow u$﻿ (essentially the reverse of following $pred$﻿ pointers from $u$﻿ back to $s$﻿
- We show $dist[u]$﻿ is as long as **this** path (and hence as long as the shortest path)
- Denote the nodes in $P'$﻿ by $v_0, v_1, ..., v_l$﻿ where $v_0 = s$﻿ and $v_l = u$﻿
- Again, let $L_j = w(v_0 \rightarrow v_1 \rightarrow ... \rightarrow v_j)$﻿
- We prove by induction that $\forall_{j>0}: dist[v_j] = L_j$﻿
- **base:** **$dist[v_0] = dist[s] = 0 = L_0$**﻿
- **inductive step:** suppose $\forall_{j>0}: dist[v_{j-1}] = L_{j-1}$﻿
    - When we set $pred[v_j] = v_{j-1}$﻿, we set $dist[v_j] = dist[v_{j-1}] + w(v_{j-1}, v_j)$﻿
    - By I.H. $dist[v_j] = L_{j-1} + w(v_{j-1}, v_j)$﻿
    - By definition: $L_j = L_{j-1} + w(v_{j-1}, v_j)$﻿
    - So, $dist[v_j] = L_j$﻿
- So, what does this mean? We have proved that $dist[v_j]$﻿ = length of a **particular path** **$P'$**﻿ **in the graph**. Length of $P'$﻿ is ≥ the length of the **shortest path**. Thus, $dist[v_j]$﻿ ≤ shortest path $s \rightsquigarrow u$﻿

  

In the above, we shown that $dist[u]$﻿ is ≤ and ≥ the shortest $s \rightsquigarrow u$﻿ path, hence, it is equal to it. So, what is the run-time of Dijkstra’s Algorithm? I’ll copy the algorithm below for reference.

```Python
def Dijkstra(adj[1...n], s):
	pred[1...n] = [null, null, ..., null]
	dist[1...n] = [infty, infty, ..., infty]
	pq = new priority queue
	
	dist[s] = 0
	for u = 1...n:
		pq.enqueue(u, dist[u]) # [node, priority]

	while pq is not empty:
		u = pq.dequeueMin()
		for v in adj[u]:
			if dist[u] + w(u,v) < dist[v]:
				dist[v] = dist[u] + w(u,v)
				pred[v] = u
				pq.changePriority(v, dist[v])
	
	return pred, dist
```

- Initializing the arrays and priority queue → $O(n)$﻿
- Each `enqueue` is $O(logn)$﻿ complexity. As we are doing this for all nodes → $O(nlogn)$﻿
- For the `while` loop, just like BFS, we consider its complexity together with the `for` loop. We will `pq.dequeMin()` n times → $O(nlogn)$﻿
- Inside the `for` loop, we will perform this for every edge in the graph, and since `pq.changePriority` is $O(logn)$﻿, this results in a total complexity of $O(mlogn)$﻿
- So, the algorithms **total time is** **$O((n+m)logn)$**﻿

To obtain the actual shortest path $s \rightsquigarrow t$﻿, we just inspect `pred[t]`

- If it is `NULL`, there is no such path
- Otherwise, just follow `pred` pointers back to `s`

---

## Bellman-Ford

This is a **single-source** shortest path algorithm in a graph with **possibly negative** edge weights, but no **negative cycles.**

**→ Quick Note**

For any subsequent shortest-path algorithm, they will solve shortest path problems as long as there are no cycles having negative weight. If a graph has a negative cycle, there is no shortest path, and there are no known efficient algorithms that can find the shortest path in these graphs.

→ **End Note**

  

**→ Bellman-Ford**

The bellman-ford algorithm solves the single source shortest path problem in any **directed** graph without negative weight cycles. This algorithm is very simple to describe:

- Repeat $n-1$﻿ times: relax every edge in the graph (where relax is the updating step in Dijkstra’s algorithm).
- So, what is the runtime analysis of this algorithm?
- Array initialization the beginning is $O(n)$﻿
- For the nested `for` loops:
    - **outer** loop → $O(n)$﻿ outer iterations
    - **inner** loop → $O(m)$﻿ inner iterations per outer iteration
        - $O(1)$﻿ work inside this
- Total runtime is $O(mn)$﻿ → which could be $O(n^3)$﻿ where number of edges is $n^2$﻿ (every node is connected to every other node)

```Python
def BellmanFord(n, E[1...m], s):
	pred[1...n] = [null, null, ..., null]
	D[1...n] = [infty, infty, ..., infty]
	D[s] = 0

	for i = 1 ... n:
		for (u,v,w) in E:
			if D[u] + w < D[v]:
				D[v] = D[u] + w
				pred[v] = u

	return (D, pred)
```

  

[https://www.youtube.com/watch?v=obWXjtg0L64](https://www.youtube.com/watch?v=obWXjtg0L64)

This video provides a good visualization of the algorithm and why it works

You may notice, sometimes, we may have already found the optimal distances (after an iteration, no relaxations are made as graph is already maximally relaxed), but the other `for` loop is still needlessly looping. To counter this, we can optimize this algorithm by implementing early stopping:

```Python
def BellmanFord(n, E[1...m], s):
	pred[1...n] = [null, null, ..., null]
	D[1...n] = [infty, infty, ..., infty]
	D[s] = 0

	for i = 1 ... n:
		changed = False
		for (u,v,w) in E:
			if D[u] + w < D[v]:
				D[v] = D[u] + w
				pred[v] = u
				changed = True

		if not changed:
			break
		if i == n:
			return NEGATIVE_CYCLE 

	return (D, pred)
```

**→ Best Case Execution**

Say, the graph looks like the first row :

![[Screenshot_2023-11-20_at_12.58.14_PM.png]]

So, say we get really lucky, and the edges are selected in **left to right** order. So, we would finish finding the shortest path in one **outer loop** iteration. Then, after the second, we realize nothing additional is being relaxed → quit algorithm.

  

**→ Worst Case Execution**

Now, consider the same graph:

![[Screenshot_2023-11-20_at_1.12.44_PM.png]]

Now, imagine that we are unlucky. So, when we choose edges, it is from **right to left** order. Obviously, it takes **many more** iterations to find the minimal distances.