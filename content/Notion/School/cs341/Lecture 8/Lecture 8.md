## DP Solution to 0-1 Knapsack

Previously, we looked at how we can solve 0-1 knapsack problems. Consider this instance:

- I have a bag of 4 items, as shown below:

![[Screenshot_2023-10-11_at_12.00.31_PM.png]]

![[Screenshot_2023-10-11_at_12.00.44_PM.png]]

What are the slides saying? We have 4 items and we want to maximize the total value given a weight constraint. So, how can we do this? We can consider 2 instances:

1. Item 4 is **NOT** added
    1. In which case, we want to find maximum value of items 1-3 given the weight limit of 7
2. Item 4 **IS** added
    1. In which case, we need to find the maximum value of items 1-3 given the weight limit of $7 - w_{item4}$﻿ = $6$﻿

  

With this, we can form a recurrence relation and generalize this to just an item $i$﻿. The following chart summarizes this idea well:

![[Screenshot_2023-10-11_at_12.04.00_PM.png]]

We especially take notice of the bottom 2 requirements, being: $i \ge 2$﻿ and $m \ge w_i$﻿. But what about the other cases? Those form our edge and base cases:

1. **General Case** **$i \ge 2$**﻿ **and** **$m \ge w_i$**﻿
    1. Since the weight limit is greater than how much item i weighs, **we can carry item i**, and recurse on the rest:
    2. $P[i,m] = max\{P[i-1, m], p_i + P[i-1, m-w_i]\}$﻿
2. **Special Case 1:** **$i \ge 2$**﻿ **and** **$m \lt w_i$**﻿
    1. In this case, adding the item would go over the weight limit, so we can only recurse on the option of not adding it
    2. $P[i,m] = P[i-1, m]$﻿
3. **Special Case 2:** **$i = 1$**﻿ **and** **$m \ge w_i$**﻿
    1. This is a base case. If we only have one item, and it is under the weight limit, we will simply take it. No recursing as there is only one item
    2. $P[i,m] = p_i$﻿
4. **Special Case 3:** **$i = 1$**﻿ **and** **$m \lt w_i$**﻿
    1. In this case, the weight limit is above this item, so we cannot take it. It is also one item, so in this case, we just don’t take it
    2. $P[i,m] = 0$﻿

  

This generates the following final recurrence:

![[Screenshot_2023-10-11_at_12.29.37_PM.png]]

So, now, we want to construct the bottom-up table. The following slides demonstrate this:

![[Screenshot_2023-10-11_at_1.38.35_PM.png]]

![[Screenshot_2023-10-11_at_1.38.56_PM.png]]

![[Screenshot_2023-10-11_at_1.39.15_PM.png]]

![[Screenshot_2023-10-11_at_1.39.30_PM.png]]

**⇒ Quick Summary of the Slides**

- Given the the recurrence relation, we realize we only ever need the previous row to compute the current row.
- The first couple of rows will always be 0, as we can’t have anything when the weight of that item is greater than the weight limit
- As the previous row is the only dependency, it does not matter in which order we populate a given row (from 0 → m, or from m → 0)

  

Here is the pseudo-code for the algorithm:

```Python
def Knapsack01(p[1...n], w[1...n], M):
	P = new table[1...n][0...m]

	# base cases where i=1
	for m = 0 ... M:
		if m < w[1]:
			P[1][m] = 0 # for this base case, we cannot fit the item into bag 
		else:
			P[1][m] = p[1]

	# for general cases where i >= 2
	for i = 2 ... n:
		for m = 0 ... M:
			if m < w[i]:
				P[i][m] = P[i-1][m]
			else:
				P[i][m] = max(P[i-1][m], p[i] + P[i-1][m-w[i]])
	
	return P[n][M] # this is the optimal profit. what if we want the optimal items that get the optimal solution? 
```

  

To get the contents of the most optimal profit, we trace back from this position in the table. Here is the lecture video (check time 34:18):

[https://www.youtube.com/watch?v=TYgwTo1hQ0Q&ab_channel=trbot](https://www.youtube.com/watch?v=TYgwTo1hQ0Q&ab_channel=trbot)

![[Screenshot_2023-10-11_at_2.18.08_PM.png]]

Essentially, consider the above filled in table. We can see the maximum profit available is 18. The position above is is less ($17 < 18$﻿), so, we need to take item 6. Taking item 6 means we have $30 - 16 = 14$﻿ weight left, so we go to weight limit 14 for item 5’s row. There we see that position has the same value as the one above it, meaning item 5, given the 14 weight limit, is not needed to obtain the maximum profit. So, we wil not add item 5, and move up to item 4. The same logic repeats until we have a list of which items to add. The solution is a list of binary options, 1 indicating the item was chosen, and 0 indicating otherwise. The pseudo-code is as follows:

```Python
def Knapsack01_Items(p[1...n], w[1...n], M, P): # P is the table 
	sol = new Array[1...n] # this is the list of binary indicators 
	i = n
	m = M
	# we start at P[n][M] as we know this is where the max profit is

	while i > 1: # we are stating from item 6, by how how the table has been constructed 
		if P[i][m] == P[i-1][m]: # where the position is same as above, --> this item is not needed 
			sol[i] = 0
			i = i - 1
		else:
			sol[i] = 1
			m = m - w[i]
			i = i - 1

	sol[1] = (P[i][m] > 0) ? 1 : 0
	return sol
```

So, whats the runtime of this algorithm? → $\Theta(n)$﻿

**⇒ Complexity of the Algorithm**

Let’s assume the unit cost model, so additions / subtractions take constant time $O(1)$﻿. The table takes $\Theta(nM)$﻿ to construct. Consider the following slide:

![[Screenshot_2023-10-11_at_2.53.54_PM.png]]

At the moment, our recurrence relation has 4 paths it can take, with there being multiple base cases. We can look to simplify this, which will lead to cleaner code and make the recurrence simpler.

![[Screenshot_2023-10-11_at_3.02.38_PM.png]]

  

## Coin Changing Problem

- **Instance:**
    - A list of **coin denominations**, $1 = d_1, d_2, ..., d_n$﻿, and a positive integer $T$﻿, which is called the **target sum .**
    - Note $d_1$﻿ is 1, as every currency needs this denomination. The rest are arbitrary
- **Find:**
    - An n-tuple of non-negative integers, say $A = [a_1, a_2, ..., a_n]$﻿, such that $T = \sum_{i=1}^{n} a_id_i$﻿ and such that $N = \sum_{i=1}^{n} a_i$﻿ is minimized.
    - So, minimize the number of coins used

So, from this, what subproblems should we consider? How should we construct our bottom-up table?

> Now, this is slightly different than the previous example of the 0-1 knapsack problem, we are only choosing one of each item, potentially. So, we would demonstrate this with a binary variable 0/1. But now, in this problem, we can > 1 of each coin denomination, which changes how we solve it.

![[Screenshot_2023-10-11_at_4.50.07_PM.png]]

So, the above shows the recurrence relation we arrive at. Also, the above shows that this problem has optimal substructure. With the recurrence relation, we can now start constructing our bottom-up table. Consider the following:

![[Screenshot_2023-10-11_at_4.55.43_PM.png]]

![[Screenshot_2023-10-11_at_4.55.58_PM.png]]

As seen, the formulation process is very similar to what we did for the 0-1 knapsack problem. We construct a 2d grid, where we often have the area of interest as the y-axis, and the constraint as the x-axis (**keep this in mind, helpful for visualization**). Based on the recurrence formula, we only depend on the row above us and nothing else. The above also includes the pseudo-code. Let’s further analyze the code:

- We have defined 2 tables, $N$﻿ and $J$﻿
    - Each cell in $N$﻿ is the minimum number of coins needed to fulfill that target sum $t$﻿ using only $1...i$﻿ coin denominations.
    - Each cell in $J$﻿ ⇒ $J[i,t]$﻿ is the **number of coins** of type $d_i$﻿ used in $N[i, t]$﻿
        - This is useful for tracking what number of each denomination was used

  

With the $J$﻿ table, if we wanted to, could construct the count for each of the coin denominations from $1 ... n$﻿. Here is the pseudo-code:

```Python
def CoinChangingDP_coins(d[1...n], J[1...n][0...T]):
	counts = new array[1...n]
	t = T
	for i = n ... 1:
		counts[i] = J[i][t] # recall that from the above, we have made J[i][t] the number of d_i used for target t 
		t = t - counts[i]*d[i]
	
	return counts
```

  

So, what is the time complexity of `CoinChangingDP(d[1...n],T)`? For this problem, the **unit cost** computational model is reasonable here. The analysis is pasted here for simplicity:

![[Screenshot_2023-10-11_at_8.04.40_PM.png]]

So, does this algorithm run in polynomial time? Well what do we mean by this? We’ve mentioned before some things are “polynomial time” while others are not. Here is a formal definition of what we are talking about:

- An algorithm runs in (worst case) **polynomial time** IFF its runtime $R(I)$﻿ on every input is upper bounded by a polynomial in the input size $S$﻿
    - $R(I) \in O(c_0 + c_1S + c_2S^2 + c_3S^3 + ... + c_kS^k)$﻿ **for constants k and** **$c_0 ... c_k$**﻿
    - So, the above time complexity analysis shows that $R(I) \in O(nT^2)$﻿. So now we ask, is this runtime polynomial time in our input size of $S$﻿

  

So, what is the input size? → $S = bits(T) + bits(d_1) + ... + bits(d_n)$﻿

- It takes $\lceil \log_2T \rceil $﻿ **bits** to store T
- It takes $\lceil \log_2{d_i} \rceil $﻿ **bits** to store **each** **$d_i$**﻿

Assume that $d_i \le T$﻿ **(otherwise,** **$d_i$**﻿ **cannot be used at all, and should be omitted from the input)**

- Then we have $\lceil \log_2 d_i \rceil \in O(\log T)$﻿
- So, $S \in O(n\log T)$﻿

  

Now, let’s compare the runtime to the size of the input. We know that $S \in O(n\log T)$﻿ and $R(I) \in O(nT^2)$﻿

![[Screenshot_2023-10-11_at_8.31.03_PM.png]]