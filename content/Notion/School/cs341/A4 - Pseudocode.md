### Question 1:

```Python
def MinNumAirports(G=(A,B,E)):
	construct G' from bipartite graph G -> O(n+m)
	# run a modified Ford-Fulkerson algorithm, in which it returns the max flow graph, as well as its corresponding residual graph
	F, R_f = FordFulkerson(G') -> O(km)
	
	# we use the R_f to find set S that has maximum capacity, do so with with BFS from source node s, add all reachable nodes from S
	S = BFS(R_f, s) -> O(n+m)
	
	minAirportsMachines = set()
	# can apply könig's theorem to find min vertex cover
	for v in A U B: -> O(n) iterations 
		if v in A and v not in S: -> O(1)
			minAirporsMachines.add(v) -> O(1)
		elif v in B and v in S: -> O(1)
			minAirporsMachines.add(v) -> O(1)

	return minAirportsMachines

# we assume we are working with sets implemented via hash tables 
-> Total Run Time: O(n+m) + O(km) + O(n+m) + O(n) ∈ O(n+m) + O(nm) + O(n+m) + O(n) ∈ O(nm)
```

### Question 2:

```Python
def FindOptimalPath(A):
	# we construct a graph G by iterating through every 2 vertices 
	V = set()
	E = set()
	for i = 1 ... n: -> O(n) iterations
		for j = 1 ... n: -> O(n) iterations
			s1 = str(A[i]) -> O(k)
			s2 = str(A[j]) -> O(k)
			if s1 and s2 differ by exactly one digit: -> O(k) with below being O(1)
				add A[i] to V 
				add A[j] to V 
				add (s1,s2) to E with edge weight 2A[j] - A[i]
				add (s2,s1) to E with edge weight 2A[i] - A[j]

	# now, we have constructed the graph G = (V,E). Now, we run Bellman-Ford algorithm, with n = |V| and starting node as A[1] 
	dist, pred = BellmanFord(len(V), E, A[1]) -> O(n(n^2 + kn)) (explained below)
	
	# with the pred array -> traverse backwards to obtain the path
	st = stack()
	sol = []
	curr = n
		# will break out when we reach node with no pred (cur == NULL)
		while curr: -> O(n)
			if dist[curr] is infinity:
				return IMPOSSIBLE
			push A[curr] onto the st stack 
			curr = pred[curr]

	while st not empty: -> O(n)
		sol.append(st.pop())

	return sol				

-> Total Run Time: O(kn^2) + O(n(n^2 + kn)) + O(n) + O(n) ∈ O(n^3 + kn^2)
```

  

### Question 3:

```Python
def MinTrainPath(w):
	D0 = [1...n][1...n][2]
	# instead of simply copying weight matrix for base case -> need to iterate through 
	for i = 1 ... n: -> O(n) iterations
		for j = 1 ... n: -> O(n) iterations
			# these are the 2 sub cases seen in the base case
			D0[i][j][0] = w[i,j] -> below in this loop is constant work
			if w[i,j] is infinity 
				D0[i][j][0] = infinity
			else:
				D0[i][j][1] = 0

	D1 = [1...n][1...n][2]
	Dlast = pointer to D0
	DCurr = pointer to D1
	
	# for all interior nodes 
	for k = 1 ... n: -> O(n) iterations
		for i = 1 ... n: -> O(n) iterations
				for j = 1 ... n: -> O(n) iterations
					Dcurr[i,j,0] = min(Dlast[i,j,0], Dlast[i,k,0] + Dlast[k,j,0]) -> constant work
					Dcurr[i,j,1] = min(Dlast[i,j,1], Dlast[i,k,0] + Dlast[k,j,1], Dlast[i,k,1] + Dlast[k,j,0]) -> constant work
		temp = DLast
		Dlast = Dcurr
		Dcurr = temp

	# construct solution array 
	Dn = [1...n][1...n]
	for i = 1 ... n: -> O(n) iterations
		for j = 1 ... n: -> O(n) iterations
				# we look at Dlast, as it swapped over at the very end
			Dn[i][j] = Dlast[i][j][1] -> constant work

	return Dn
-> Total Run Time: O(n^2) + O(n^3) + O(n^2) ∈ O(n^3)
```

  

  

### Question 4:

```Python
def MonotonicBelmanFord(G=(V,E;w),n,m,s):
	pred[1...n] = [null, null, ..., null]
	D[1...n] = [infty, infty, ..., infty]
	prevEdgeWeight[1...n] = [null, null, .., null]
	D[s] = 0

	for i = 1 ... n: -> O(n) iterations
		for (u,v,w) in E: -> O(m) iterations
			if D[u] + w < D[v]:
					if w > prevEdgeWeight[u] or prevEdgeWeight[u] is null:
					D[v] = D[u] + w
					pred[v] = u
					prevEdgeWeight[v] = w
	
	return (D, pred)
-> Total Run Time: O(nm)
```