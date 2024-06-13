### Part 1:

Say we name this problem $\text{Adaptor-Ring}$﻿. We prove that $\text{Adaptor-Ring}$﻿ is NP-Complete. First, we show $\text{Adaptor-Ring}$﻿ is in NP. For this, we consider the input to be an array of adaptors $[a_1, a_2, ..., a_n]$﻿ where each adaptor is a pair of connecter types $[l_i, r_i]$﻿. Let $\text{Adaptor-Ring}$﻿ be the decision problem that determines whether there is some subset of adaptors such that each connector type is used once.

**→ Showing** **$\text{Adaptor-Ring}$**﻿ **is NP**

Data type of certificate is an array of adaptors, where each adaptor can be represented by some integer $1,2,...n$﻿. The desired yes-certificate is an array of adaptors $C$﻿, where each connector type is used only once and that when connected, form a ring, and each entry is connected to the one before it and the one after it. Now, we construct a poly-time `verify(I,C)` algorithm:

```Python
def verify(I = (A,t), C):
	# A is array of adaptors, t is the number of connector types 
	# we first verify that each connecter type was used only once (we should count exactly 2 adaptors having the connector type)
	type_count = map() #
	for l,r in C: 
		if l not in type_count:
			type_count[l] = 1
		else:
			type_count += 1

		if r not in type_count:
			type_count[r] = 1
		else:
			type_count += 1

	for key in type_count:
		if type_count[key] != 2:
			return False
		
	if |type_count| != t: return False

	# now we check for ring -> each adaptor 
	for a_i, a_j in A:
		if a_i[1] != a_j[0]: return False          # for adjacent pair in A, we check if the r of the left adaptor = the l of the right adaptor, needed for ring

		if A[-1][1] != A[0][0]: return False       # for the special case, very last adaptor in A and first, need to connect to form the ring

	return True
```

- The runtime of this is fairly straightforward:
    - We first iterate over $C$﻿ → so this is $|C|$﻿ iterations, where the size of $C$﻿ is at most $n $﻿ → So, at most $O(n)$﻿ iterations
    - We then iterate over `type_count` array. The maximum number of types is the just the number of adaptors → so, also $O(n)$﻿ iterations
    - Finally, we iterate over $A$﻿, which has size $n$﻿ ($n$﻿ adaptors) → so this is also $O(n)$﻿ iterations
- The input size is $I$﻿ which has $n$﻿ elements in $A$﻿, and $t$﻿ which is a constant. As mentioned above, $|C|$﻿ has size at most $n$﻿
- Evidently, the certificate-verification algorithm runtime is polynomial on the input size

  

Now, we prove correctness through 2 cases:

**→ Case 1: Let** **$I$**﻿ **be any yes-instance → find** **$C$**﻿ **such that** `**verify(I,C) = True**`

Suppose $I$﻿ is a yes-instance. That is, in this list of adaptors, there is a subset $C$﻿ where every connector-type is used and the adaptors form a ring. Running `verify(I,C)`, we won’t return false in the second for loop, as $C$﻿ is a ring of adaptors, each connection type is only present on exactly 2 adaptors. Also, as $C$﻿ uses all connector-types, it uses all $t$﻿ types. The next part checks if $C$﻿ is ring, as every 2 adaptors in $C$﻿ share a connection type, we won’t return false here. Thus, reach the end and true is returned.

**→ Case 2: If** `**verify(I,C)**` **is true → prove** **$I$**﻿ **is a yes-instance**

Suppose the certificate-verification algorithm returns true. So, this means we have some ordered array of adaptors $C$﻿, where:

- every adjacent pair in the array share a connector type, with the last and first adaptors connecting, thus forming a ring
- every connector is used once, as there are $t$﻿ connector types in $C$﻿
- Thus, $C$﻿ proves the existence of a subset of adaptors where every connector-type is used exactly once and a ring is formed.

Next, we show that this is NP-Complete by providing a polynomial transformation from the known NP-Complete problem, $\text{Undirected-Hamiltonian-Cycle Problem}$﻿. We first provide a transformation:

- Consider the input to be the graph $G=(V,E)$﻿ that contains an undirected hamiltonian cycle. Now, for every in graph $G$﻿, we interpret the nodes to be the set of all connector types, and every edge to be an adaptor. We iterate over every edge and convert it into an entry $a_i$﻿ of the array $A$﻿, where each $a_i$﻿ consists of $[l_1,r_1]$﻿ which represent the connector types (end nodes) of each adaptor (edge).
- This requires $O(|E|) = O(m)$﻿ ruintime
- The input size is the graph $G$﻿, for which the size of is $|E| = m$﻿. Hence it is evident that this transformation has polynomial runtime on the input size.
- This polynomial transformation applies a transformation to the input, and **a single call** to the oracle without modification

**→ We argue this transformation satisfies 2 key conditions:**

- → If **$I \in \mathcal{J}_{yes}(\text{\text{Undirected-Hamiltonian-Cycle}})$**﻿ then $f(I) \in \mathcal{J}_{yes}(\text{Adaptor-Ring})$﻿
    - Suppose $I$﻿ is a yes-instance of the UHC problem. So, $I$﻿ is a graph $G$﻿ that contains a hamiltonian cycle. $f(I)$﻿ is an array of edges, where each edge has 2 nodes $u,v$﻿. Since $I$﻿ contains a hamiltonian cycle, there are no edges visited more than once, and so, are a subset of the edges in $f(I)$﻿. Also, the hamiltonian cycle visits each vertex once, meaning each connector type is used only once. These edges form a cycle, which means there is some subset $C$﻿ of $f(I)$﻿, where the edges form a cycle. Thus, there is some subset of edges $C$﻿ of $f(I)$﻿ that form a ring, where each connector type is used once → hence $f(I) \in \mathcal{J}_{yes}(\text{Adaptor-Ring})$﻿
- → If $f(I) \in \mathcal{J}_{yes}(\text{Adaptor-Ring})$﻿ then $I \in \mathcal{J}_{yes}(\text{\text{Undirected-Hamiltonian-Cycle}})$﻿
    - Suppose $f(I)$﻿ is an instance of the adaptor ring problem. That is, there is some subset of adaptors $a_i = [l_i,r_i]$﻿ in $f(I)$﻿, where they form a ring, with each connector typed used once. Given this is a yes instance, there exists some subset of $f(I)$﻿ $[a_{k-1}, a_k, a_{k+1}, a_{k+2},...a_t]$﻿ such that: $r_t = l_{k-1}, r_{k-1} = l_k, r_k = l_{k+1}, ..., r_{t-1} = l_t$﻿. Construct a graph, where each $a_i$﻿ is converted into an edge, and $l_i,r_i$﻿ are converted into nodes of the graph. Since the above characteristic is present in $f(I)$﻿, it also exists in this new graph. That is, there is some set of edges $[e_1,e_2,...e_k]$﻿ that are a ring, where each edge $e_i = (u_i,v_i)$﻿ has 2 end vertices, such that $v_k = u_1, v_1 = u_2, ..., v_{k-1} = u_k$﻿. As every connector type is only used once, every pair $[v_{i-1},u_i]$﻿ is used only once → in terms of the graph, each vertex is visited only once. So, we have a sequence of adaptors (edges) that visits every vertex exactly once. Thus, this graph is instance $I$﻿, and since it contains a cycle that visits each vertex exactly once, it has a hamiltonian cycle → $I \in \mathcal{J}_{yes}(\text{\text{Undirected-Hamiltonian-Cycle}})$﻿

Therefore, as we proved that $\text{Adaptor-Ring}$﻿ is in NP and that there is a valid polynomial transformation from a NP-Complete problem, we conclude that $\text{Adaptor-Ring}$﻿ is also **NP-Complete**.

---

### Part 2:

We claim there is a poly-time algorithm that solves this problem. The idea is:

- Like the reverse-transformation in the above problem, we construct a graph from the set of adaptors
- We’re looking for a path where all adaptors are used → looking for a Eulerian path
- The euclidean path problem can be solved in **polynomial time**

Consider the following pseudo-code:

```Python
def ChainAllAdaptorsSolver(A):
	construct graph G from A where all adaptors are edges and all connector types are nodes  
	return EulerianPathSolver(G)
```

**→ Proof of Correctness:**

We provide now a proof of correctness. The first step of the algorithm is to map the array of adaptors to a graph, where each vertex represents a connector type and each edge represents an adaptor. The algorithm is attempting to find a chain that uses all adaptors. Notice that in terms of the graph, this is equivalent to looking for a path where each edge is used once. This is the exact input needed for the euclidean path algorithm, which given a connected graph, is able to determine whether a path that visits each edge exactly once exists.

- Note, in the graph $G$﻿ is comprised of multiple component graphs, then the `EulerianPathSolver` returns false. In the context of finding a chain of all adaptors, multiple component graphs means there exist some adaptors with connector types no other adaptor has, and so, can never be connected together to form a link.
- In cases, where there are multiple adaptors of the same type (say multiple adaptors of type $\text{USB-C}$﻿ to $\text{USB-A}$﻿), the algorithm is able to handle this → not a concern
- Hence, we make a modification/transformation to the input and call `EulerianPathSolver` and return’s its return value

**→ Runtime:**

The given input is $A$﻿, which is the array of $n$﻿ adaptors. Assuming the graph $G$﻿ is matrix representation, $O(n^2)$﻿ is an upperbound on the construction time. Since each of the $n$﻿ adaptors is mapped to an edge, the size of the graph is $|E| = n$﻿. We know `EulerianPathSolver` is polynomial on its input size $n$﻿. Hence, it is evident that the runtime of `ChainAllAdaptorsSolver` is also polynomial on the input size.