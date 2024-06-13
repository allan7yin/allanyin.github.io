### Part 1:

We’re looking to provide a poly-time Turing reduction of the form: $\text{Seating-Plan} \le_P^T \text{Ham-Path}$﻿, where $\text{Ham-Path}$﻿ is the problem of finding a simple path in an undirected graph that visits each vertex. We _assume the input_ _$\text{Seating-Plan}$_﻿ _is in graph format._ That is, a graph $G=(V,E) $﻿ where each vertex represents a student, and and edge $uv$﻿ represents student $u$﻿ knows student $v$﻿. We choose an adjacency matrix as the format for storing this graph, where $\text{adjMatrix}[i][j]$﻿ is 1 if student $i$﻿ knows student $j$﻿, and 0 otherwise. We want to find a permutation where adjacent students in a row do not know each other. We construct $H$﻿ from $G$﻿, where $H = (V,F)$﻿ and $v_iv_j \in F \iff v_iv_j \notin E$﻿. In $H$﻿ the edges now connect 2 students who **do not** know each other.

  

That is, a path that uses every vertex represents a permutation of students, where adjacent students to now know each other → $\text{Ham-Path}$﻿ solves this in constant time for us. Assume $\text{Ham-Path}$﻿ returns a path in the format of an array. So, we can construct the following pseudocode:

```Python
def Seating-Plan-Solver(G=(V,E)):
	construct inverse graph H from G                    # this is O(n^2)
	path_permutation = Ham-Path-Solver(H)               # we assume the oracle runs in O(1)
	
	if |path_permutation| == 0: return NO_PERMUTATION 
	return path_permutation
```

**→ Proof of Correctness**

- We now argue proof of correctness, that is, the reduction is correct. The algorithm takes the graph $G$﻿ as input. In this representation, we have edges between 2 students who know each other. So, we know these 2 students cannot sit beside each other in hallway, How can me model this as path problem?
- We want to formulate this as a path problem, but we do not want to work with a graph where edges represent 2 students **knowing** each other. Hence, we construct the inverse graph $H$﻿. Now, we have a graph with the same vertices, but now, edges represent 2 students not knowing each other.
- Now, we want to have the 2 endpoints of each edge sitting beside each other, but this is a row, so each student can only have 2 neighbours. This maps very well to finding a path in $H$﻿. For a path $v_1, v_2, ..., v_n$﻿ in $H$﻿, the edges between $v_{i-1}, v_i, v_{i+1}$﻿ represent that $v_i$﻿ knows neither $v_{i-1}$﻿ nor $v_{i+1}$﻿. This is the format the permutation output wants to be in. So, for all nodes in this path, they do not know their neighbours in $G$﻿ → hence a suitable assignment of students
- If `Ham-Path-Solver(H)` returns an empty array, then this means there is no hamiltonian path in $H$﻿. This tells us, there is no way to configure the students in a row where neighbours do not know each other, which is possible when the number of “knowledges” is too large. Likewise, by returning an array representing a path $[v_1, v_2, ..., v_n]$﻿, this represents a valid assignment of students. As every edge in this path $v_i \rightarrow v_{i+1}$﻿ is 2 students who **do not** know each other.
- Hence, this is a correct reduction. To justify it is a **polynomial** reduction, we show that the run-time on the input-size is polynomial.

**→ Runtime**

For this problem, we consider the input size to be $O(|V|^2) = O(n^2)$﻿. Constructing the inverse graph $H$﻿ is an $O(n^2)$﻿ operation since we are working with the adjacency-matrix representation (iterate over all rows and columns). We also made the assumption the oracle ($\text{Ham-Path-Solver}$﻿) runs in constant time. Thus, it is evident the `Seating-Plan-Solver` runs in poly-time.

### Part 2:

This is **false.** The class of **NP Problems** are a family of decision problems. This problem returns a permutation of students, instead of a **yes** or **no** solution.

### Part 3:

This is **false**. A problem is $\Pi \in$﻿ NP-Hard does not need to be a decision problem and does not need to be in NP (as answered in the above). Even if we did assume that we had a correct poly-time Turing reduction from $\text{Ham-Path}$﻿ to $\text{Seating-Plan}$﻿ of the form $\text{Ham-Path} \le_P^T \text{Seating-Plan} $﻿, $\text{Ham-Path}$﻿ is **not** NP-Complete, as it is not a decision problem. Therefore, this does not hold, and given this reduction, we cannot conclude this $\text{Seating-Plan} $﻿ is NP-Hard.