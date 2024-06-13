Recall last time we saw an algorithm that solved the **greedy interval selection** problem. Now, we will look to prove it is indeed the optimal solution through contradiction:

### Optimality Proof — for greedy interval selection

Before anything, heres a nice video that shows the same proof, but for exchange:

[https://www.youtube.com/watch?v=hVhOeaONg1Y&ab_channel=BackToBackSWE](https://www.youtube.com/watch?v=hVhOeaONg1Y&ab_channel=BackToBackSWE)

Consider an input $A[1...n]$﻿

- Let **G** be the greedy solution
- Let **O** be the optimal solution
- “Greedy stays ahead of optimal” argument
    - Intuition: out of a given set of intervals, greedy picks **as many as optimal**

![[Screenshot_2023-10-07_at_11.46.35_AM.png]]

**NOTE: We are not assuming the optimal solution has the same sort order.** By assuming the existence of an optimal solution O that is different from the greedy solution G, and then reordering O to match G, you create a situation where both solutions look the same in terms of order or structure:

- the reordering helps in comparing the structural properties of the two solutions.
- It simplifies the comparison between G and O by aligning them in a way that makes it easier to analyze and identify differences or contradictions.

  

**⇒ Proof by contradiction**

After we have re-ordered the optimal solution, we can imagine something like this:

![[Screenshot_2023-10-07_at_4.55.07_PM.png]]

In which both are now ordered by increasing time. This structure helps use visualizes a proof. Let the function $f(G_1)$﻿ represent the finishing time of the $G_1$﻿ block and let the function $S(G_1)$﻿ represent the starting time of the $G_1$﻿ block. We first want to prove the lemma:

  

![[Screenshot_2023-10-07_at_4.58.41_PM.png]]

![[Screenshot_2023-10-07_at_4.58.49_PM.png]]

The base case is true by definition (since Greedy choses the interval with the earliest finish time). Here is how we operate the inductive step:

- **assume** **$f(G_{i-1}) \le f(O'_{i-1})$**﻿
    - Since $O'$﻿ is feasible, we know $f(O'_{i-1}) \le s(O'_{i})$﻿
    - so, $f(G_{i-1}) \le s(O'_i)$﻿

So, the finish time of the $i-1$﻿ interval for the greedy solution is less than the start time of the $i$﻿ interval of the optimal solution. What can we deduce from this?

![[Screenshot_2023-10-07_at_5.04.49_PM.png]]

Observe that, after the $i-1$﻿ interval of greedy, we choose the next interval to be the one with the smallest finishing time. So, this interval has to be smaller than the finishing time of the $i $﻿ interval of optimal, since it can also just choose the $i$﻿ interval of optimal (as that interval is disjoint from the $i-1$﻿ interval of greedy). So, from $f(G_{i-1}) \le s(O'_i)$﻿ of above, we know:

- $f(G_i) \le f(O'_i)$﻿

  

With this lemma, we can construct a proof by contradiction.

![[Screenshot_2023-10-07_at_5.09.32_PM.png]]

So, the optimal solution cannot have more than what we have in our greedy solution, which for this problem, proves the greedy solution is indeed optimal.

  

## Knapsack Problem

Another optimization problem we can solve with a greedy algorithm is the classic **knapsack problem**. Given a set of items, each with a weight and a value, determine the maximum value that can be obtained by selecting a subset of the items that fit into a knapsack of limited capacity. We can define some definitions first:

- **Instance:**
    - **profits:** **$P[p_1, p_2, ..., p_n]$**﻿
    - **weights:** **$W[w_1, w_2, ..., w_n]$**﻿
    - **capacity:** **$M$**﻿
    - The above are all positive integers
- **Feasible Solution:**
    - An n-tuple $X = [x_1, x_2, ..., x_n]$﻿ where $\sum_{i=1}^{n} w_ix_i \le M$﻿. That is, the total weights of all chosen options is less than the constraint $M$﻿.

  

A special instance of the **knapsack problem** is the 0-1 Knapsack problem. In the 0-1 Knapsack problem (often just denoted as the Knapsack problem), we require that $x_i \in \{0,1\}$﻿ with $1 \le i \le n$﻿. In the ==**rational**== **knapsack problem**, we require that $x_i \in Q$﻿ and $0 \le x_i \le 1, 1 \le i \le n$﻿. For this model, we are looking to find a **feasible solution: A feasible solution** **$X$**﻿ **tha maximizes** **$\sum_{i=1}^{n} p_ix_i$**﻿.

  

So, what is the difference between the standard knapsack problem and the **rational knapsack problem?** Here are the differences:

1. **Normal Knapsack Problem:**
    - **Objective:** Maximize the total value of selected items.
    - **Constraint:** The sum of the weights of selected items should not exceed the knapsack's capacity.
    - **Decision:** Choose whether to include or exclude each item (binary decision: 0 for exclude, 1 for include).
2. **Rational Knapsack Problem:**
    - **Objective:** Maximize the total value-to-weight ratio of selected items.
    - **Constraint:** The sum of the weights of selected items should not exceed the knapsack's capacity.
    - **Decision:** Choose whether to include or exclude each item (binary decision: 0 for exclude, 1 for include).

  

Here is the optimal greedy algorithm for the rational knapsack problem:

**⇒ Strategy:** consider items in decreasing order of profit divided by weight (i.e. we maximize local evaluation criteria $p_i/w_i$﻿)

- **profits:** **$P = [20,50,100]$**﻿
- **weights:** **$W = [10,20,100]$**﻿
- **weight limit:** **$M = 10$**﻿

So, profit divided by weight is: $\frac{P}{W} = [2, 2.5, 1]$﻿. So, the algorithm selects the second item at price 25 (optimal). Here is the pseudocode for this:

```Python
def PreProcess(A[1...n], M): # A[i] = (p_i, w_i)
	sort(A) # by decreasing profit divided by weight  => Θ(nlogn)
	let p[1...n] be the profits in A # this is post sort, so in decreasing order 
	let w[1...n] be the weights in A # this is sorted according to the corresponding profits 
	return GreedyRationalKnapsack(p,w,M)

def GreedyRationalKnapsack(p[1...n],w[1...n],M):
	X = [0] * len(p) # each index is if item i was chosen, 0 or 1, or for one index, a percentage of it  => Θ(n)
	weight = 0 # current weight of the knapsack 

	for i in range(n): # iterate over all items  => Θ(n)
		if weight + w[i] > M then: # if adding the current weight is going to go over, we add what we can of this one 
			X[i] = (M - weight) / w[i]
			weight = M
			break
		else:
			X[i] = 1 # here, the weight limit will not be violated, can add the entire item to the knapsack 
			weight += w[i]
	
	return X

# Either X=(1,1,...,1,0,...,0) or X=(1,1,...,1,𝒙𝒊,0,...,0) where 𝒙𝒊 ∈ (0,1)
=> Θ(nlogn) is the total run time 
```

  

All thats left is proving this algorithm is **feasible and optimal.**

  

**⇒ Informal Feasibility Argument —** should be good enough to show feasibility on assessments

- **feasibility: all** **$x_i$**﻿ **are in** **$[0,1]$**﻿ **and the total weight is** **$\le M$**﻿
- either _everything fits_ in the knapsack, or:
    - when we exit the loop, **weight is exactly M**
    - every time we write to $x_i$﻿ its either 0, 1, or $\frac{M- weight}{w_i}$﻿ where $weight + w[i] > M$﻿
        - rearranging the latter, we get $\frac{M - weight}{w_i} < 1$﻿
        - and $weight \le M$﻿, so $\frac{M - weight}{w_i} \ge 0$﻿
        - so, we have $x_i \in [0,1]$﻿

  

We look to prove optimality of this with **exchange argument.** The "exchange argument" is a concept often used in the context of proving optimality in algorithms, particularly in the design and analysis of greedy algorithms. It is a technique employed to show that, if there exists a solution to a given problem that is not optimal, then it's possible to exchange (replace) parts of that solution with parts of another solution, leading to a new solution that is either better or equally good.

  

⇒ **Optimality - An Exchange Argument**

For simplicity, assume that the profit / weight ratios are all distinct, so:

$\frac{p_1}{w_1} > \frac{p_2}{w_2} > \frac{p_3}{w_3} > ... \frac{p_n}{w_n}$

Suppose the greedy solution is $X = (x_1, x_2, ..., x_n)$﻿ and the optimal solution is $Y = (y_1, y_2, ..., y_n)$﻿. We will prove that $X = Y$﻿, where $x_i = y_i$﻿ for all $1 \le i \le n$﻿. For the sake of contradiction, suppose $X \ne Y$﻿. If they are not equal, there must exist some index $j$﻿ such that $x_j \ne y_j$﻿ (identical up until this index).

![[Screenshot_2023-10-08_at_1.38.47_PM.png]]

![[Screenshot_2023-10-08_at_1.40.01_PM.png]]

What the above 4 slides show is that, with how we have defined the greedy algorithm, if there is an index where the $x_j \ne y_j$﻿, then $y_j$﻿ cannot be more, by definition. So, there is some other index $k$﻿, such that $j < k$﻿, where we have some weight allocated for it. By moving that weight to $j$﻿, as item $j$﻿ is worth more (per unit of weight) than item k, even replacing a tiny amount of item $k$﻿ will improve the solution, which is a **contradiction**, as we assumed that $Y$﻿ was the optimal solution. So, therefore, $X = Y$﻿, and we have proved optimality.

![[Screenshot_2023-10-08_at_1.45.30_PM.png]]

![[Screenshot_2023-10-08_at_1.51.30_PM.png]]

- Last but not least, we show we still abide by the weight constraint, so showing $weight(Y') \le M$﻿. Recall that changing from $Y $﻿ to $Y'$﻿, the amount of item $k$﻿ we’re removing and item $j$﻿ we are adding are the same in weight, meaning total weight has not changed, and thus, $weight(Y') = weight(Y) \le M$﻿ → feasible
- Now, we just show that how $Y'$﻿ is better than $Y$﻿

$profit(Y') = profit(Y) + \frac{\delta}{w_j}p_j - \frac{\delta}{w_k}p_k \\$

We know that $\frac{p_j}{w_j} > \frac{p_k}{w_k}$﻿, which means if $\delta > 0$﻿, then $\delta(\frac{p_j}{w_j} - \frac{p_k}{w_k}) > 0$﻿, and as we can choose a $\delta > 0$﻿, $profit(Y') > profit(Y)$﻿

  

### Optimality Proof Without Distinctness

**The above has assumed that the profit-weight ratios were unique. What if they are not?**

In this case, we can have multiple optimal solutions.

- **Key Idea:** Let $Y$﻿ be an optimal solution that matches $X$﻿ on a maximal number of indices
- **Observe:** if $X$﻿ is really optimal, then $Y = X$﻿

Suppose for the sake of contradiction, that $Y \ne X$﻿, we will modify $Y$﻿ preserving its optimality, but making it match $X$﻿ on **one more index** (a contradiction)

![[Screenshot_2023-10-08_at_5.37.35_PM.png]]

![[Screenshot_2023-10-08_at_5.53.37_PM.png]]

In the above, let $j$﻿ be the first index where the solutions differ. So, we know that $y_i < x_i$﻿ as by definition of greedy algorithm, we always take as much as we can from the higher in value one. So, this means we know that there has to be some $k$﻿, such that $k > j$﻿ and $y_k > x_k$﻿, as the overall weight of the 2 solutions must be the same, so the $y_i < x_i$﻿ needs to be compensated somewhere else. So, we need to move $\delta$﻿ amount of weight, either to make $y_j = x_j$﻿ or $y_k = x_k$﻿, which will provide the contradiction we are looking for.

  

In the bottom right corner slide above, we are letting $\delta = w_k(y_k - x_k)$﻿, meaning we are looking to make $y_k = x_k$﻿. So, with this, we arrive at a contradiction, as we assumed that the optimal solution $Y$﻿ matched $X$﻿ on a maximal number of indices. But, we have shown it can match $X$﻿ on one more index. Therefore, we have proved optimality and so, $Y = X$﻿.

  

![[Screenshot_2023-10-08_at_6.02.55_PM.png]]

![[Screenshot_2023-10-08_at_6.03.06_PM.png]]