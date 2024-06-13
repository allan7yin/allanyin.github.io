**Recall from last lecture, we left off the the following conclusions:**

- ==$\text{TSP-Dec} \le_P^T \text{TSP-Optimal Value}$==﻿
- ==$\text{TSP-Dec} \le_P^T \text{TSP-Optimization}$==﻿
- ==$\text{TSP-Optimal Value} \le_P^T \text{TSP-Dec}$==﻿

This lecture, we will also show that ==$\text{TSP-Optimization} \le_P^T \text{TSP-Dec}$==﻿==, as well as introduce the complexity class== **==NP==**

---

### Reducing TSP-Optimization to TSP-Dec

Recall that TSP-Optimization returns the actual Hamiltonian cycle in the graph with path weight. We already know how to get the minimum weight **T** of the minimum HC after showing $\text{TSP-Optimal Value} \le_P^T \text{TSP-Dec}$﻿ can be implemented with Binary Search ⇒ use **T** along with calls to the oracle to somehow figure out **which edges are involved in the minimum HC**

  

The algorithm idea is, we remove each edge in the graph, and then immediately call `TSP-Dec` to see if a hamiltonian cycle still exists that is ≤ the minimum path-weight hamiltonian cycle in the graph. If after removing an edge, `TSP-Dec` is now false, this tell us that edge was part of the path → and so we add it to solution set. If `TSP-Dec` still returns `True`, this means the removed edge was not in the path, and so, we keep moving.

![[Screenshot_2023-12-03_at_9.36.21_AM.png]]

- First line calls `TSP-OptimalValue-Solver` as we know this is poly-time reducible on `TSP-Dec-Solver`
- `H` is initially empty set, $w_0$﻿ is copy of edge weights
- Iterate over all edges:
    - If removing the edge (setting its weight to $\infin$﻿ in $w_0$﻿) results in `TSP-Dec-Solver` is `False`, then we add the edge to `H` and its corresponding edge weight to $w_0$﻿ by doing $w_0[e] = w[e]$﻿
- The, we return `H`, the minimum path weight HC

So we return `H`, a HC in the graph (could be more, but it is **one** solution). How to prove correctness?

  

**→ Correctness**

- **Loop Invariant:** there exists a HC of weight T* in $w_0$﻿ (loop invriants holds before and after the loop)
- `TSP-OptimalValue-Solver` returns T*, which is the minimum path weight for a HC in the graph
- So since weight of `H` is T* → it is minimum HC
    - How do we prove this? We know H will **contain** a HC → but could it contain extra edges?
    - A quick proof by contradiction solves this:
        
        ![[Screenshot_2023-12-03_at_9.54.16_AM.png]]
        
    - See the above: **assume for the sake of contradiction, that H is NOT precisely C**
    - So, there are extra edges → say $e$﻿ is such an edge
    - So, since we did not remove it in the algorithm, it must be removing it would result in a HC of weight T* to not be possible in the graph at that time. So, if we remove $e$﻿, then HC $C$﻿ would not exist, which is a contradiction since $e$﻿ is not in $C$﻿, so even if we did remove $e$﻿, $C$﻿ would still be in the current graph → thus, **$C$**﻿ **is exactly** **$H$**﻿
- Also the above general explanation of this algorithm

  

So, correctness is true, **runtime?**

![[Screenshot_2023-12-03_at_9.58.28_AM.png]]

![[Screenshot_2023-12-03_at_9.58.52_AM.png]]

Some quick notes about the run-time analysis:

- The size of the input in this case is being measured in the **unit-cost model**
- The lower bound on the input size is the number of edges $Size(I) = \Omega(m)$﻿
- So, if we look at the runtime which is: $\text{poly(Size(I)) + } O(m)$﻿
    - We know that $O(m) \in O(Size(I)^1)$﻿ → so $O(m) $﻿ is $poly(Size(I))$﻿
    - So, 2 $Poly(Size(I))$﻿ added together means **total runtime is** **$poly(Size(I))$**﻿
- So this is a polytime reduction

---

### Complexity Class NP

**→ Example: Subset-sum problem**

- Suppose we are given some integers -7, -3, -2, 5, 8
- Does **some** subset of these **sun to 0?**
    - In this case yes → (-3) + (-2) + 5 = 0
    - Finding this set can be very difficult
- Suppose we are given a **certificate** consisting of an array of numbers, and claim it represents such as subset
    - A certificate, in this example, by array of numbers as a subset of the input, where the oracle claims it sums to 0
    - If this is true, it is a **yes-certificate**
    - Can we use a yes-certificate to solve the problem efficiently?
- Of course, the oracle could lie, and return whatever it wanted → could even be not even array of integers, anything
- Could we figure out if the oracle is lying in **polynomial time?**
    - Yes, we could always check:
        - Is array?
        - Integers?
        - Subset?
        - Sum to 0?
- **So, given a certificate, we can check whether the oracle is lying or telling the truth → both in polynomial time**
    - This is a **polynomial-time certification verification algorithm**
    - We will formally define certificates later on

  

**→ Subset-sum Via Non-deterministic Oracle**

> Note: Non-deterministic _refers to the fact that the Oracle can make arbitrary choices when providing answers_

- Suppose there is a **non-deterministic oracle**, which returns a **subset that sums to 0 (if one exists)** and otherwise, can return anything (**even garbage**).
- We call the oracle’s output a **certificate**
- Given a **certificate**, can you **verify in polytime** whether it describes a solution to the problem?

```Python
# given such an oracle, this algorithm would solve subset-sum
def SubsetSumWithOracle(I): 
	C = Oracle(I)
	return verify(I,C)

def verify(I, C):
	if C not subset of I then return false
	return (sum(C) == 0)
```

→ **Non-deterministic** is the **N** in **N**P problems, as they are named because of the oracles

→ In this question **non-deterministic** just means the oracle is magically guaranteed to return a yes-certificate if one exists