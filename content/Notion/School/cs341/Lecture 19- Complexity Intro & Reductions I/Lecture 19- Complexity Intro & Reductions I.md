Intractability is the study of the **hardness** of the problems. There are these problems that we know are very very hard, and that we do not know the polynomial time solutions to and some we can prove are impossible. We are going to spend most of our time studying the hardness of **Decision problems**, which are a class of problems

### Decision Problems

- **Decision problem:** Given a problem instance $I$﻿, answer a certain question “yes” or “no”
- **Problem instance:** input for the specified problem
- **Problem solution:** Correct answer (”yes” or “no”) for the specified problem instance.
    - $I$﻿ is a **yes-instance** if the correct answer for instance $I$﻿ is “yes”
    - $I$﻿ is a **no-instance** if the correct answer for the instance $I$﻿ is “no”
- **Size of problem instance:** **$Size(I)$**﻿ is the number of bits required to specify (or encode) the instance $I$﻿

### Complexity Class P

- **Algorithm Solving a Decision Problem:** An algorithm $A$﻿ is said to **solve** a decision problem $\Pi$﻿ provided that $A$﻿ find the correct answer (”yes” or “no”) for every instance $I$﻿ of $\Pi$﻿ in finite time
- **Polynomial-time Algorithm:** An algorithm $A$﻿ for a decision problem $\Pi$﻿ is said to be a **polynomial-time** algorithm provided that the complexity of $A$﻿ is $O(n^k)$﻿, where $k$﻿ is a positive integer and $n = Size(I)$﻿.
- **The Complexity Class P** denotes the set of all decision problems that have polynomial-time algorithms solving them. We write $\Pi \in P$﻿ if the decision problem $\Pi$﻿ is in the complexity class $P$﻿
    - Its important to note that this is **not the class of algorithms which we know the polynomial-time solutions**
    - It is the class of problems for which polynomial time algorithms exist → maybe we haven’t even discovered those solutions yet, but it does not change whether the problem belongs in $P$﻿

  

**→ To show something is in P, is by showing there exists a poly-time solution to solve it**

Let’s look back at some problems we’ve seen throughout the course, and some that we have not, that see on the surface to be quite similar, but actually have very different complexities, depending on some slight variations to the problem. Likewise, we’ll see some problems that, on the surface, may look vastly different, but in actuality, are quite similar.

  

**→ Knapsack Problems**

Consider the following slide:

![[Screenshot_2023-11-25_at_8.22.54_PM.png]]

Consider the above knapsack problems. The above is a 0-1 knapsack decision problem, where we either take an item, or we don’t → we cannot take **part** of an item. The above is similar to the **0-1 knapsack** problem we saw before, but this is **0-1 Knapsack-Dec**, which is a decision problem variation. The one below follows the same concept, but we can take fractional parts of a particular item. So, what is the relative problem hardness for these 2?

- Well, the ideas of the algorithms are very similar, so perhaps their relative hardness is similar?
- Remember that the **Rational Knapsack Problem**, we could solve efficiently with a greedy algorithm, which ran in $O(nlogn)$﻿ which is pretty good
- On the other hand, the DP solutions runtime for the **0-1 Knapsack Problem** had a run time of $O(nM)$﻿, where $M$﻿ was the weight limit in the problem. But, this algorithm is only **polynomial time** for relatively small $M$﻿. If a problem instance had small number of items (small $n$﻿), but a very large weight (say $M \sim 2^n$﻿), then this algorithm runs in **exponential time** rather than **polynomial time**
    - So, there is **no known algorithm that always solves this problem under polynomial time →** hence why this problem is known as NP-hard (more on this later)

  

![[Screenshot_2023-11-26_at_12.06.41_PM.png]]

We can look at 2 more example problems, that fall into this category → **The problem of finding a cycle in a graph**

- We are familiar with how to solve the first problem → DFS and check for back edges → polynomial time algorithm
- Now, what about a **Hamiltonian Cycle?** This is a cycle that visits every vertex exactly once → incredibly difficult task
- Maybe from a first glance, they do not look that much different. But, in actuality, the second one is **much more difficult** than the first one, exponentially so

---

So, how do we go about reasoning about the relative difficulty of these problems? How do we say one is harder than another?

### Polynomial-time Turing Reductions

> **Vocal Notes from Prof. Brown  
>   
> **How do we say that, for example, rational knapsack decision is no harder than rational knapsack? We had a solution to the rational knapsack problem, and with it, we can reduce the problem of rational knapsack decision by just running ration knapsack and checking if we are above target profit. So, once we’ve solved one, its not much harder to solver the other. What does it even mean to be **“harder”, “easier” or “about the same difficulty”?** For us in this course, we consider a problem to very hard if its exponential time and somewhat easy if it is polynomial time.
> 
> - If there is only a polynomial time complexity gap between the 2 problems (so if I can solve one of the problems in polynomial time $\Pi_2$﻿, then with some polynomial time overhead, I can solve another problem $\Pi_1$﻿, then we know $\Pi_1$﻿ is only polynomial-time more difficult than $\Pi_2$﻿
> - This is the non-formal wording. To formalize this below.

  

- Suppose $\Pi_1, \Pi_2$﻿ are problems (not necessarily decision problems). A (hypothetical) algorithm $B$﻿ to solve a sub-problem $\Pi_2$﻿ is called an **oracle** for $\Pi_2$﻿
- Suppose that $A$﻿ is an algorithm that solves $\Pi_1$﻿, assuming the existence of an oracle $B$﻿ for $\Pi_2$﻿. ($B$﻿ is used as subroutine within the algorithm $A$﻿.)
- Then we say that $A$﻿ is a **Turing Reduction** from $\Pi_1 to \Pi_2$﻿, denoted $\Pi_1 \le ^t \Pi_2$﻿
- A **Turing Reduction** **$A$**﻿ is a **polynomial-time Turing Reduction** if the running time of $A$﻿ is polynomial, under the assumption that the oracle $B$﻿ has **unit cost** running time
    - We assume the subroutine $B$﻿ takes constant time as this allows us to to analyze the complexity of the reduction from $\Pi_1$﻿ to $\Pi_2$﻿ where the run-time of $B$﻿ is not really a factor. Then, we can argue that regardless of the runtime of $B$﻿ (if its polynomial, thats fine), we will still end with polynomial time algorithm to solve $\Pi_1$﻿
- If there is a polynomial Turing Reduction from $\Pi_1$﻿ to $\Pi_2$﻿, we write: $\Pi_1 \le^T_P \Pi_2$﻿
- For example, we’ve looked at the **all-pairs-shortest-path problem**. We can reduce this into multiple **single-source shortest path algorithm calls** (say multiple BellmanFord calls). Which is to say, we are making calls to $B$﻿, our subroutine. Since $B$﻿ (BellmanFord) runs in polynomial time, so this would be polynomial Turing reduction
- **In general:**
    - A reduction is typically something like
        1. **transforms the larger problem’s input** so that it can be fed to the oracle
        2. **transforms the oracle’s output** into a solution the larger problem

  

To see this more, we consider the following problems: **Travelling Salesperson Problem**.

![[Screenshot_2023-11-26_at_1.29.12_PM.png]]

We will show polynomial-time Turing Reductions between all 3 variants of this problem, to show they are all polynomially equivalent. So, if we can solve one of them in polynomial time, when we can solve all of them in polynomial time. **Fun fact,** as computer scientists, we don’t know any polynomial time solutions to any of these problems. So, it turns out the traveling salesperson problem is an **extremely hard problem to solve → NP complete.** Let’s make some specific notes about the above each of the above problems:

1. This problem is looking for a **Hamiltonian** cycle in the graph while minimizing the sum of edge weights → in this problem, all edge weights are positive integers
2. Instead of looking for the minimum edge weight hamiltonian cycle, we are looking for the minimum edge weights sum on the hamiltonian cycle
3. The last one is the decision version of this problem → Does there exist a hamiltonian cycle with sum of edge weights less than $T$﻿?

  

Its is very easy to use polynomial-time Turing reductions to show that different versions of the **TSP** are polynomially equivalent: if one of then can be solved in polynomial time, then all of them can be solved in polynomial time. But, as above, it is believed that none of them can be solved in polynomial time).

> **Remember:  
>   
> **A polynomial-time Turing reduction means that one problem can be solved using a polynomial number of queries to an oracle that solves another problem. However, if the second problem itself has no known solution, then the reduction doesn't provide a solution for the first problem either—it just reduces the complexity class. The reduction shows that if you could solve the second problem, you could solve the first one, but it doesn't guarantee a solution exists for either. It’s important to understand this distinction, and be clear on what we the above is saying.

  

Now, we know that:

- $\text{TSP-Dec} \le^T_P \text{TSP-Optimal Value}$﻿
- $\text{TSP-Dec} \le^T_P \text{TSP-Optimization}$﻿

As if we know how to find a hamiltonian cycle in $G$﻿ with a minimum sum of edge weights, or how to to find that sum of minimum edge weights → then we would be able to answer the decision question, of wether that sum is $\le T$﻿. So from this, we know there could be some polynomial overhead between the first 2 problems, and the corresponding decision problem. **But, we are trying to show that these are all polynomially equivalent. So, in order to show this, we must also show:**

- $\text{TSP-Optimal Value} \le^T_P \text{TSP-Dec} $﻿
- $\text{TSP-Optimization} \le^T_P \text{TSP-Dec} $﻿

That is, if we have a solution to $\text{TSP-Dec} $﻿ that is $O(1)$﻿ time, then we can solve both of the above in polynomial-time

---

→ $\text{TSP-Optimal Value} \le^T_P \text{TSP-Dec} $﻿

![[Screenshot_2023-11-26_at_7.33.12_PM.png]]

So, what can we do? We are now trying to show that, if we had some **oracle** that could answer the query “is there exist a hamiltonian cycle $H$﻿ in $G$﻿ with $w(H) \lt T$﻿ in constant time, then we could find the minimum $T$﻿ such that theres is a hamiltonian cycle in the graph with $w(H) = T$﻿. Consider the graph on the left:

- So now the problem boils down to → **how should we call TSP-Dec** to solve this problem?
- On the left, we’ve tried just using an arbitrary value, 100, which returns yes → but this doesn’t really tell us anything.
- To solve this, we can:
    - Let’s start by trying $\text{TSP-Dec}(G,w,0)$﻿, where $T = 0$﻿, and we continue to increase $T$﻿ by 1 until this function returns `True` to us
    - So, how many calls are we making? How far do we need to go to conclude that we are done? Is there some point, for some very large $T$﻿, that there is **no** hamiltonian cycle in the graph, how would we know? We don’t want to make infinite calls.
    - The largest hamiltonian cycle edge weight sum is ≤ than the sum of all edge weights in the graph. So, we can stop at that value. It turns out, this is an **incredibly inefficient solution → exponential time** (the sum of edge of the weights can be exponential in the number of bits needed to write down all those weights)
    - The solution is to use binary search, with the sample pseudo-code shown below

![[Screenshot_2023-11-26_at_8.38.36_PM.png]]

The above shows how we can use binary search. We define our starting range (lo, hi) which we define as the 0 and the sum of all edge weights, respectively. If there is no hamiltonian cycle with sum of edge weights ≤ hi, then that means there is no hamiltonian cycle in the graph period → we can directly return in this case. Otherwise, we do a binary search. We define `mid` to be the middle value between `hi` and `lo`. If `TSP-Dec-Solver(G,w,mid)` returns true, that means there exists a hamiltonian cycle with $w(H) \le mid$﻿. So, we check the values from `mid` to `hi`. Otherwise, we check from `lo` to `mid`. The important question we ask is: **“Is this a polynomial-time reduction?” → Look at the last 3 grey textboxes from above.**

  

**→ Size of the input** **$I = (G,w)$**﻿

Suppose $G$﻿ is represented as an **array of adjacency lists** (one list for each vertex), with each list containing edges to neighbouring vertices, and an edge is represented by a **weight** and the name of the **target** vertex. So, each element of the adjacency list is a **pair(target node, weight of node).** We would then have:

$Size(I) = |V| + \sum_{e \in E}(log w(e) + 1 + log|V| + 1)$

- We first add $|V|$﻿ as we have an array of adjacency list’s. We are assuming that each array index is a **pointer to an adjacency list**, and we assume we can store a pointer in $O(1) $﻿ bits
- Then, we will need store something for each edge in the graph
    - The weight of the edge → in bits, it is $logw(e) + 1$﻿
    - The number of bits needed to store the target vertex (some vertex 1 to n, a.k.a |V|) → need $log|V| + 1$﻿

What would it mean to have a **runtime** **$T$**﻿ that is **polynomial in** **$Size(I)$**﻿**.** We say that $T$﻿ is **polynomial in** **$Size(I)$**﻿ (denoted $T \in poly(Size(I)$﻿) iff:

- There exists a constant $c$﻿, s.t. $T \in O(Size(I)^c)$﻿

**But wait …** **$V,E$**﻿ and $w$﻿ could be represented in many different ways. **Could the** **choice of representation** affect our complexity result? This would only be the case in very inefficient representations (that are **exponentially larger than optimal)** → We rule out such inefficient representations for the purpose of proving polynomial runtime.

  

So, we see the input size above → let’s relate it to the runtime of the above `optimal-value` function?

![[Screenshot_2023-11-26_at_9.40.45_PM.png]]

![[Screenshot_2023-11-26_at_9.56.47_PM.png]]

![[Screenshot_2023-11-26_at_9.56.58_PM.png]]

![[Screenshot_2023-11-26_at_9.58.21_PM.png]]

So, from the above, we can see that that `TSP-Optimal Value` is a polytime reduction. Now, we argue correctness. In particular, we argue it is **a correct reduction from TSP-Optimal Value to TSP-Dec.**

- **Need to prove**: `TSP-Optimal Value` returns the weight $W$﻿ of the shortest **Hamiltonian Cycle** **$H$**﻿ **in** **$G$**﻿
- **Sketch:** We return $\infin$﻿ if there is no $H$﻿. The loop invariant is $W \in [lo,hi]$﻿. So, at termination when $hi = lo$﻿, we return exactly $hi = W$﻿

  

So, `TSP-Optimal Value` is **polytime**, and is a **correct** reduction. So, we have therefore shown that **TSP-Optimal Value is polytime reducible to TSP-Dec**. So, if an $O(1)$﻿ implementation of TSP-Dec-Solver exists, then we have a polytime implementation of TSP-Optimal-Value-Solver! In fact, TSP-OptimalValue-Solver remains polytime even if the implementation of the oracle runs in polytime instead of O(1)! Why is this?

- Consider the polynomials $P_R(s)$﻿ and $P_O(s)$﻿ which are the runtimes for the reduction and oracle respectively
- The worst possible runtime occurs if every step in the reduction is a call to the oracle → we will have $P_R(s)P_O(s)$﻿
- But, multiplying polynomials with degrees $d_1, d_2$﻿ results in a polynomial with degree $\le d_1 + d_2$﻿, which means even with a polynomial time oracle, the reduction will still be polynomial time

![[Screenshot_2023-11-26_at_10.28.44_PM.png]]

The above provides some useful choices for input size based on the kind of input. Any of the options is fine, and should be selected specific to the problem: perhaps in some problems, we may only need or case about the vertices, and vice versa.