### DFS Application: Testing Whether a Graph is a DAG

**Recall the definition:** a directed graph $G$﻿ is a directed acyclic graph (DAG), if $G$﻿ contains no directed cycle

> **Lemma 6.7:** a directed graph is a DAG iff a DFS search encounters no back edges

This lemma is very easy to understad why this is. We quickly show the proof for both directions:

- **→ Any back edge creates a directed cycle**
    - A back edge $u \rightarrow v$﻿ where $v$﻿ is an ancestor of $u$﻿ means we have now created a cycle → QED
- **← Suppose there exists a directed cycle, show there exists a back edge**
    - So if here is a directed cycle, there is some cycle of the form $v_1 \rightarrow v_2 \rightarrow ... \rightarrow v_k \rightarrow v_1$﻿. WLOG, let $v_1$﻿ be the earliest discovered node in the cycle. So, by DFS we know that $d[v_1] < d[v_k] < f[v_k] < f[v_1]$﻿. This then means that the edge type is **back edge →** QED

  

So, with this lemma, let’s construct an algorithm. But how will we know we have encountered a back edge in the algorithm? As we saw in the previous lecture, if we edge $u \rightarrow v$﻿, then it is back edge if $v$﻿ is **gray**. Here is the pseudo-code for this:

```Python
global variables:
	pred[1...n] = [null, null, ..., null]
	colour[1...n] = [white, white, ..., white]
	d[1...n] = [0,0,...,0]
	f[1...n] = [0,0,...,0]
	time = 0
	DAG = True

# recall DFSVisit from previous lecture 
def DFSVisit(adj[1...n], v):
	colour[v] = gray
	time = time + 1
	d[v] = time

	for each w in adj[v]:
		if colour[w] == white:
			pred[w] = v
			DFSVisit(w)
		# we make the followig modification here -> if the colour of w is gray, we know we have a back edge -> set DAG = False
		if colour[w] == gray:
			DAG = False

	colour[v] = black
	time = time + 1
	f[v] = time 

def IsDag(adj[1...n]):
	for v = 1 ... n:
		if colour[v] == white:
			DFSVisit(adj, v)
	return DAG
```

![[Screenshot_2023-11-18_at_9.33.22_PM.png]]

---

### Topological Sort: Finding node orderings that satisfy given constraint

The idea of this is, given a DAG, we can arrange the nodes a order, where each node will have directed edges with nodes to the right of it. The below demonstrates this:

![[Screenshot_2023-11-18_at_9.58.31_PM.png]]

![[Screenshot_2023-11-18_at_9.58.39_PM.png]]

> **Formal Definition:** A directed graph $G = (V,E)$﻿ has a **topological ordering**, or **topological sort**, if there is a linear ordering < of all the vertices in $V$﻿ such that $u \lt v$﻿ whenever $(u,v) \in E$﻿

> **Lemma 6.5:** a DAG contains a vertex of indegree 0

How to prove this? Suppose for the sake of contradction, that we have a DAG in which each vertex has positive indegree. So, say we have something like $v_{n+1} \rightarrow v_n \rightarrow ... \rightarrow v_3 \rightarrow v_2 \rightarrow v_1$﻿. We could go on an on, but by pidgeon hole principle, at some point, one of these must be repeated, as $v_n+1$﻿ needs to have some node directed towards it, and it has to be on the $n$﻿ nodes to the right of it. Hence, a contradiction → there is at least one vertex of indegree 0.

  

So, how do we perform a topological sort with DFS? The finishing times are very important for this, and yield the following lemme:

> **Lemma 6.8:** suppose $D$﻿ is a DAG. Then $f[v] < f[u]$﻿ for every arc $uv$﻿

How do we know this? From the table we’ve seen before. You can reason this without it as well.

> **Theorem:** If $D$﻿ is a DAG, and we order the vertices in **reverse order of finishing time (largest to smallest finish time)**, then we get topolocial ordering.

How to prove? This is demonstated in the diagram below:

![[Screenshot_2023-11-18_at_11.25.10_PM.png]]

So, here is the pseudo-code we will follow:

```Python
global variables:
	pred[1...n] = [null, null, ..., null]
	colour[1...n] = [white, white, ..., white]
	d[1...n] = [0,0,...,0]
	f[1...n] = [0,0,...,0]
	time = 0
	DAG = True

def TopologicalSort(adj[1...n]):
	S = new stack
	for v in 1 ... n:
		if colour[v] == white:
			DFSVisit(adj,v,S)
	if DAG then return S
	return null

def DFSVisit(adj[1...n], v):
	colour[v] = gray
	time = time + 1
	d[v] = time

	for each w in adj[v]:
		if colour[w] == white:
			pred[w] = v
			DFSVisit(w)
		if colour[w] == gray:
			DAG = False

	colour[v] = black
	# we add this to our DFSVisit -> this will ensure that first ones pushed are first to finish
	# so the final stack after DFS, we can just pop and add to an array and we have descending order, as desired 
	S.push(v)
	time = time + 1
	f[v] = time 
```

**!! FOR SLIDE 11 → TRY AND SEE IF WE START ON NODE 3, WHAT HAPPENS? DOES TOPOLIGICAL SORTING STILL APPLY? IM ASSUMING YES**

---

### Strongly Connected Components

In the previous lecture, we looked at connected graphs. Recall that connected graphs are ones where:

- A graph $G$﻿ is **strongly connected** iff every node is reachable from every other node. More fomally → $\forall_{w,v} \exists w \rightsquigarrow v$﻿

Now, consider the following graph images:

![[Screenshot_2023-11-18_at_11.43.46_PM.png]]

![[Screenshot_2023-11-18_at_11.44.08_PM.png]]

We’re interested in whether we can take a graph, that maybe as a whole isn’t strongly connected, but we could divide it in some way, such that after the division, the components we divided into are each strongly connected. So, the above left one has 2 scc. What about the top right one? The graph can also be divided into 3 components, where each one is strongly connected. **We want our SCCs to be maximal (as large as possible).** So, the only correct solution for this graph is the top left division. So, what is this useful for?

  

**→ Applications of SCCs and Component Graphs**

![[Screenshot_2023-11-19_at_10.13.19_AM.png]]

- Finding **all cyclic** dependencies in code
    - Can find **single** cycle with an easier DFS-based algorithm
    - But it is nicer to find **all** cycles at once, so you don’t have to fix one to expose another
- Another very important application of strongly connected components is **Data filtering →** before running other algorithms that is more extensive that the algorithm for finding strongly connected componenets
- Imagine that we are Google and we want to implement Google Maps, so we’re looking at a large map; nodes = intersections, edges = roads
- We don’t really want to run an expensive path-finding algorithm for shortest paths from a source to a destination on the entire global graph. Ideally, we would like to filter and crop to some **local graph** (subgraph) to run the algorithm on. So, we may first outline a rectangle of the area we are interested in
    - Then find the maximal SCC containing the source and target

  

---

### Component Graphs

Consider the following graph, with its SCCs circled:

![[Screenshot_2023-11-19_at_10.30.30_AM.png]]

![[Screenshot_2023-11-19_at_10.40.34_AM.png]]

Now, from this graph, we can define a **component graph →** which has one node from each connected component. The above right image is the component graph. **Can there be a cycle in the component graph? → NO,** if there are paths both ways between the components, then they are the same SCC. If there are paths both ways, then for every node in component 1, there is a path to get to every node in component 2, and vic versa → The component graph is **DAG**.

  

Recall, to determine if a graph strongly connected we ran an algorithm that:

- Picked an arbitrary node → ran DFS to explore everyything to check if we can reach every node from it
- Then we reversed all the edges and ran DFS again

So what if we did that here? Observe the following slides:

![[Screenshot_2023-11-19_at_11.27.50_AM.png]]

![[Screenshot_2023-11-19_at_11.28.07_AM.png]]

We’ll first look the top left slide. Simply running the DFS on the graph results in a couple of problems

- If we start on node a, we will visit ndoes d, b, c, e, f, g. However, nodes e,f,g are not the same SCC as node a. We want our DFS to stay in one SCC when visiting a particular node.
- The same issue arrises when we reverse the edges. Now, when we do `DFSVisit(h)`, we will also visit nodes j,l,k, which is a separate SCC. We can make some observations, however:
    - If we visit node i before node h → we will visit component l,j,k in its entirety. Then, call `DFSVisit(h)` and visit that component → will not result in 2 components being explored at once
    - In other words, for a particular SCC $c$﻿, other reachable SCC’s should be **visited first**, so that when we visit the nodes of $c$﻿, we stay in component $c$﻿ as the reachable SCC’s have already beedn visited.

  

It turns out, the finishing time computed in the forward direction (from simply calling `DFS` on the graph) will help us in determining the order to run `DFS` on the reversed graph. This will ensure this property of reachable SCC’s visited first, is satisfied. So, the top right slide shows how we perform this:

- We first compute the `DFS` on the graph, helping us compute the finishing times for each node
    - By nature of DFS, in the graph, the node with the largest finishing time is the node capable of reaching the most number of other nodes
- By reversing the edges, that particular node is still capable of reaching the most number of other nodes. But now, this node cannot reach other components (if it could before) as the direction of the edge connecting its component with another component is now **reversed**
- So, in the reversed graph, we visit the available node with the largest finishing time each time.
    - In the graph above (right) we would:
        - visit node j → visits l,j,k component
        - visit node h → visits h,i component
        - visit node a → visits a,b,c,d component
        - visit node e → visits e,g,f component
    - Each component is visited separately, as desired

```Python
def SCC(adj[1...n]):
	DFS(adj) # computes the finishing times for all nodes 
	let order[1...n] = node labels sorted by largest to smallest finish time
	
	reverse all edges in adj
	
	colour[1...n] = [white, white, ..., white]
	comp[1...n] = [0,0,...,0] # component array -> label every node with which component they belong to
	for i = 1 ...n:
		v = order[i]
		if colour[v] == white:
			scc = scc + 1
			SCCVisit(adj, v, scc, colour, comp) # modified DFSVisit

	return comp

def SCCVisit(adj[1...n],v, scc, colour, comp)
	colour[v] = gray
	comp[v] = scc
	
	for each w in adj[v]:
		if colour[w] == white:
			SCCVisit(w)
	
	colour[v] = black
```

What is the time complexity?

- First DFS is $O(n+m)$﻿
- Reversing all edges is $O(n+m)$﻿
- Initializing arrays is $O(n)$﻿
- As we’ve seen already, in the `for` loop → total work being done is $O(n+m)$﻿
- So, the total run time is $O(m+n)$﻿

  

**→ Correctness**

How to prove correctness for this algorithm? → Each top level call to `SCCVisit` explores excatly the nodes in **one SCC.** The proof hinges on a key lemma about the finish times of SCCs in the component graph.

- **Key Definition:** For a strongly connected component $C$﻿, we let
    - the discovery time of the component = discovery time of the first node (smallest discovery time of a node in the component)
    - the finish time of the component = finish time of last node (largest finish time of a node in the component)
    - $d[C] = min\{d[v]: v\in C\}$﻿ and $f[C] = max\{f[v]: v\in C\}$﻿
- **Key Lemma:** If $C_i, C_j$﻿ are SCCs and there is an edge $C_i \rightarrow C_j$﻿ in $G$﻿, then $f[C_i] > f[C_j]$﻿ → this makes sense, nodes in $C_i$﻿ can read those in $C_j$﻿ and so, will have larger finishing times and earlier discovery times. The proof for this is fairly straightforward:
    - **Case 1:** **$d[C_i] < d[C_j]$**﻿
        - In this case, we know we are first exploring $C_i$﻿ then $C_j$﻿. So, this means that we will need to finish $C_j$﻿ and return to $C_i$﻿ in order for $C_i$﻿ to finish. Thus → $f[C_i] > f[C_j]$﻿
    - **Case2:** **$d[C_i] > d[C_j]$**﻿
        - In this case, we are first exploring $C_j$﻿ then $C_i$﻿. Recall that the over-arching graph is a DAG, which means $C_i$﻿ and $C_j$﻿ are also DAGs. Thus, there is no edge from $C_j \rightarrow C_i$﻿, as this would produce a cycle. So, $C_j$﻿ is fully explored (and finished) before we explore $C_i$﻿. Thus → $f[C_i] > f[C_j]$﻿.

So, now that we have introduced this **key definition** and proved the **key lemma**, we now complete the proof of correctness for `SCC` and `SCCVisit`.

- Consider the first top-level `SCCVisit(u)`
- Let $C$﻿ be the SCC contaning $u$﻿ and $C'$﻿ be any other SCC
- We know that since we are calling `SCCVisit` on the node with largest finish time → $f[C] > f[C']$﻿
- By the above lemma, _**if there were an edge**_ _**$C' \rightarrow C$**_﻿_**, then we would have**_ _**$f[C'] > f[C]$**_﻿_**.**_ So, since we have the opposite of $f[C'] > f[C]$﻿, we know there is **no** edge $C' \rightarrow C$﻿.
- Now, consider the reverse graph. Since there is no edge $C' \rightarrow C$﻿ in original graph, then there is no edge $C \rightarrow C'$﻿ in the reversed graph. So what does this mean?
    - `SCCVisit(u)` in the graph cannot visit any other SCC $C'$﻿ (remeber $u \in C$﻿). So, we’re going to precisely explore the nodes in this SCC, $C$﻿.
- Now this is only for the first top-level call. What about the rest?
    
    - Similar argument for the subsequent **top-level** calls to `SCCVisit`
    - In $G$﻿, the overall graph, edges go from larger to smaller finishing times. In $H$﻿ (the reversed graph), **edges go from smaller to larger finishing times**, and a larger finishing time means that node is already explored.
    - So, each top-level call explores **exactly** one SCC, and sets `comp[v] = scc` for all nodes in the SCC