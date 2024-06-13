## Finishing D&C and Greedy Algorithms 1

### Closest Pair Problem

⇒ **Input: Set P of n 2D points**

**⇒ Output: pair p and q s.t. dist(p,q) minimum over all pairs**

**⇒ Breaks ties arbitrarily**

**⇒** **$dist(p,q) = \sqrt {(p.x - q.x)^2 + (p.y - q.y)^2}$**﻿

  

One approach is to use divide and conquer. The logic is as follows:

**⇒ Divide the space into 2 halves, call them** **$L$**﻿ **and** **$R$**﻿

1. $(p,q)$﻿ both in $L$﻿
2. $(p,q)$﻿ both in $R$﻿
3. One of $(p,q)$﻿ in $L$﻿ and the other in $R$﻿ (we call thisa **spanning pair**)

  

Here is the pseudo code for the above:

```C
def ClosestPair(P[1...n])
	sort(P) by x values 
	Recurse(P)

def Recurse(P(1...n]) // pre-condition is that P sorted by X
	// base case
	if n < 4 then compare all pairs and return closest
	
	// divide and conquer step 
	pairL = Recurse(P(1...(n/2)])
	pairR = Recurse(P((n/2) + 1 ...n])

	// combine 
	δ = min(dist(pairL, dist(pairR))
	pairS = findMinSpanningPair(δ, P) // we want to compute this as effeciently as possible. How? 
	return minDistPair(pairL, pairR, pairS)
```

So, we need an effecient way to find the minimum spanning pair. Since we have the smallest pair on the left, and the smallest pair on the right, we don’t actually need to go through every point again looking for the spanning pair. We can make the following observation:

  

**Observation 1**

- let $\delta = min(dist(pair_L), dist(pair_R))$﻿
- So, we need only to look $\delta$﻿ left an $\delta$﻿ right of the dividing line. This is because, we were to look over it, then those spanning pairs instantly have a larger distance compared to $pair_L$﻿ and $pair_R$﻿. So, we need only to look if a spanning pair exists within this range. The below diagrams illustrate this nicely:

![[Screenshot_2023-10-03_at_12.30.21_PM.png]]

![[Screenshot_2023-10-03_at_12.30.36_PM.png]]

  

**Observation 2**

![[Screenshot_2023-10-03_at_2.17.35_PM.png]]

So, why do we need only to look in that square? We do this since any point outside of the square autmoatically means the distance between the 2 points is $\gt \delta$﻿. While some points within the square might also make this so, its just **easier** to work with a $\delta$﻿ x $\delta$﻿ square.

  

This leads us to the general core idea for finding the closest pair:

1. Start from the **lowest y value point** in the strip
2. Search the $\delta$﻿ x $\delta$﻿ square points on the opposite side
3. Repeat steps 1 and 2 for the **next lowest y valued point**

  

Will look something like:

![[Screenshot_2023-10-03_at_2.25.50_PM.png]]

![[Screenshot_2023-10-03_at_2.26.06_PM.png]]

While this opposite switching is ideal, in practice, it yields little to no additional benefit and it unnecessarily complicates code. Instead of looking in the opposite $\delta$﻿ x $\delta$﻿ square, we will look at the current $\delta$﻿ x $2\delta$﻿ rectangle on top of the point.

![[Screenshot_2023-10-03_at_2.28.51_PM.png]]

![[Screenshot_2023-10-03_at_2.29.05_PM.png]]

So, we now show the pseudocode for the `findMinSpanningPair(P)` from the above:

```C
def findMinSpanningPair(δ, P([1...n]) // P is sorted by x  
	S = { p in P: abs(P[n/2] - p.x) <= δ } // filters this into only points within δ of the dividing line => Θ(n)
	sort(S) // do this so that p.y is increasing order  => Θ(nlogn)
	if len(S) < 2 return (-∞, -∞), (∞, ∞) // only 1 or 0 points in S, not enough to form pair
	minPair = (S[1], S[2]) // arbitrary point to start at 
	
	for i = 1 ... len(S)
		for j = (i+1) ... len(S)
			if S[j].y - S[i].y > δ then break // too big 
			minPair = minDistPair(minPair, (S[i], S[j])) => Θ(1)

	return minPair 
```

  

Now, we see that we have a 2 inner for loops, which may raise some concerns over the above being $\theta(n^2)$﻿ complexity. **However, we claim, and prove, that the inner loop only performs** **$O(1)$**﻿ **operations.**

- Closely examine the inner loop. Notice that we are at most iterating as many times as there are points in the $\delta$﻿ x $2\delta$﻿ rectangle. So, how many points can be in that rectangle? → As many as there are in the left square plus the right square
- Recall that when we defined $\delta$﻿, we defined as the smallest distance between 2 points in either $L $﻿ or in $R$﻿. So, this means in a $\delta$﻿ x $\delta$﻿ square, we cannot have 2 points within it that are not on the border of the box. The description below echoes this idea:

![[Screenshot_2023-10-03_at_3.00.28_PM.png]]

So, in the $\delta$﻿ x $2\delta$﻿ rectangle, we can haver **at most 8** points. What does this ential → `j-loop` performs at most 8 iterations, where each one is constant time work, meaning the entire loop does $\theta(1) $﻿ work. So, the `i-loop` does $\theta(n)$﻿ work, meaning the entire `findMinSpanningPair(δ, P)` function runs in $\theta(nlogn)$﻿ time. Now that we have the cost associated with all helper functions, we can now formulate the recurence relation and determine the runtime of the `ClosestPair` function (we paste the code again for convenience):

```C
def ClosestPair(P[1...n])
	sort(P) by x values  => Θ(nlogn)
	Recurse(P) => T(n)

def Recurse(P(1...n]) // pre-condition is that P sorted by X
	// base case
	if n < 4 then compare all pairs and return closest
	
	// divide and conquer step 
	pairL = Recurse(P(1...(n/2)]) => Θ(n) + T(n/2)
	pairR = Recurse(P((n/2) + 1 ...n]) => Θ(n) + T(n/2)

	// combine 
	δ = min(dist(pairL, dist(pairR))
	pairS = findMinSpanningPair(δ, P) // we want to compute this as effeciently as possible. How? => Θ(nlogn)
	return minDistPair(pairL, pairR, pairS)
```

$1. \;T'(n) = ClosestPair(P(1...n]) \\$

**⇒ The above is a derivation of the runtime (note that we get 4 from using recursion tree method)**

So, our final runtime is: $T'(n) = \Theta(nlog^2n)$﻿

  

**⇒ We can improve the above run time if we also sort the points by y-value**

In the above implementation of `findMinSpanningPair` , we saw that we had to sort by y-value to ensure we had them in increasing order. Sorting by these pairs is what makes `findMinSpanningPair` an $\Theta(nlogn)$﻿ operation. To better optimize this, we can create a copy of the points and sort them by y-vlaue. This way, we need only to pass the sorted y-values to `findMinSpanningPair`. Here is how we could re-write it with pre-sorted y-value points:

```C
def ClosestPair(P[1...n])
	Px = sort(P) by increasing x values  => Θ(nlogn)
	Py = sort(P) by increasing y values  => Θ(nlogn)
	Recurse(Px, Py) => T(n)

def Recurse(Px[1...n], Py[1....n]) // pre-condition is that P sorted by X
	// base case
	if n < 4 then compare all pairs and return closest
	
	// divide and conquer step 
	xmid = Px[n/2].x
	PxL = Px[1...(n/2)] // x <= mid => Θ(n)
	PxR = Px[(n/2) + 1 ... n] // x > mid => Θ(n)
	PyL = select p from Py where p.x <= mid => Θ(n)
	PyR = select p from Py where p.x > mid => Θ(n)
	pairL = Recurse(PxL, PyL) => T(n/2)
	pairR = Recurse(PxR, PyR) => T(n/2)

	// combine 
	δ = min(dist(pairL), dist(pairR))
	pairS = findMinSpanningPair(δ, Py, xmid) // we want to compute this as effeciently as possible. How? => Θ(nlogn)
	return minDistPair(pairL, pairR, pairS)


def findMinSpanningPair(δ, Py[1...n], xmid) // Py is sorted by y
	S = { p in P: abs(xmid - p.x) <= δ } // filters this into only points within δ of the dividing line => Θ(n)
	if len(S) < 2 return (-∞, -∞), (∞, ∞) // only 1 or 0 points in S, not enough to form pair
	minPair = (S[1], S[2]) // arbitrary point to start at 
	
	for i = 1 ... len(S)
		for j = (i+1) ... len(S)
			if S[j].y - S[i].y > δ then break // too big 
			minPair = minDistPair(minPair, (S[i], S[j])) => Θ(1)

	return minPair 
```

Now, in the above, `PxL` and `PyL` contain the same points, just sorted in different order. Now, we don’t need to recurse. So, this brings our **total run time to** **$\Theta(nlogn)$**﻿ **as opposed to** **$\Theta(nlog^2n)$**﻿

  

---

## Greedy Algorithms

Greedy algorithms are techniques we use to solve optimization problems. Here is some familiar terminology from CO250:

- **Problem:** given a problem instance, find a feasible solution that maximizes (or minimizes) a certain object function
- **Problem Input:** input for the problem
- **Problem Constraints:** Requirements that must be satisfied
- **Feasible Solution:** Set of all outputs that satisfy the constraints
- **Optimal Solution:** the feasible solution such that the objective function value is maximizes (minimized)

  

We continue by defining some terminology for **greedy algorithms:**

- **partial solutions:**
    - for a problem instance $I$﻿, it should be possible to write a feasible solution $X$﻿ as a tuple $[x_1, x_2, ..., x_n]$﻿ for some integer $n$﻿, where $x_i \in X$﻿ for all $i$﻿.
    - A tuple $[x_1, x_2, ..., x_i]$﻿ where $i < n$﻿ is a **partial solution** if no constraints are violated
    - In other words, this is the solution we are constructing at each turn in the greedy algorithm
- **choice set:**
    - for a partial solution $X = [x_1, x_2, ..., x_i]$﻿ where $i < n$﻿, we define the **choice set:**
        - $choice(X) = \{y \in X: [x_1, x_2, ..., x_i, y]$﻿ is a partial solution $\}$﻿
    - in other words, this refers to the set of options or candidates from which the algorithm can choose at each step. The algorithm makes decisions by selecting one element from the choice set at each iteration, effectively adding it to the partial solution
- **local evaluation criterion:**
    - For any $y \in X, g(y)$﻿ is a **local evaluation criterion** that measures the cost or profit of including $y$﻿ in a partial solution
    - In other words, it is the current constraint we must meet at the current iteration. Also, local tells us that **we cannot consider future choices** when deciding whether to include y in our (partial) solution
- **extension:**
    - given a partial solution $X = [x_1, x_2, ..., x_i]$﻿ , choose $y \in choice(X)$﻿ so that $g(y)$﻿ is as small (or large) as possible. Update $X$﻿ to be the (i+1)-tuple $[x_1, x_2, ..., x_i, y]$﻿
    - in other words, this is just adding the best “choice” at the current step, and adding it to our partial solution

  

So, in essence, a greedy algorithm is one that starts with an “empty” partial solution, and repeatedly extends it until a feasible solution $X$﻿ is constructed. This feasible solution may or may not, be optimal.

- No backtracking
- No looking ahead to consider future consequences of current action
- Often consist of a pre-processing step
- **correctness:** for greedy algorithms, it is possible to prove that they always yield optimal solutions. However, these proofs can be tricky and complicated.

  

We take our first look at greedy algorithms with an example:

  

### Interval Selection Problem

- **Input:** a set of $A = \{A_1, A_2, ..., A_n\}$﻿ of time intervals. Each interval $A_i$﻿ has a start time $s_i$﻿ and a finish time of $f_i$﻿
- **Feasible Solution:** a subset $X$﻿ of $A$﻿ containing **pairwise disjoint** intervals (the intervals do not overlap)
- **Output:** a feasible solution of **maximum size** (one that maximizes $|X|$﻿)
- A classic example of the interval selection problem is the activity selection problem, where each interval represents an activity, and the objective is to schedule the maximum number of non-overlapping activities.

  

To start, we want to first sort all time intervals by their $f_i$﻿ in increasing order. At any stage, we choose the earliest starting interval that is disjoint from all previously chosen intervals.

- **Partial solutions:**
    - $X = [x_1, x_2, ..., x_i]$﻿ where each $x_i$﻿ is an interval for the output
- **Choices:**
    - $Y = A $﻿ (i.e all intervals)
    - $Choice(X) = \{y \in Y : [x_1, x_2, ..., x_n]$﻿ respects all constraints $\}$﻿
        - where $y \notin X$﻿ and
        - $\forall_{x \in X}$﻿ $disjoint(y,x)$﻿
- **Local Evaluation Function:**
    - $g(y) = s_j$﻿ where $y = A[j]$﻿
    - i.e $g(y) =$﻿ start time of interval y

  

Here are 3 attempts at a solution, 2 of which are flawed:

- Sort by the starting time in increasing order, and choose the earliest starting time at each stage
- Sort by duration of the in increasing order, and choose the smallest one at each stage
- Sort by ending time in increasing order, and choose the earliest finishing interval at each stage

  

![[Screenshot_2023-10-05_at_4.21.53_PM.png]]

![[Screenshot_2023-10-05_at_4.22.11_PM.png]]

**⇒ Strategy 3**

This one is where we sort by ending time in increasing order, and choose the earliest finishing interval at each stage. Here is the pseudo-code for this:

```C
def GreedyIntervalSelection(A[1...n])
	sort(A) by increasing finish times  
	X = [A[1]]
	prev = 1        // index of last selected interval 
	
	for i = 2...n
		if A[i].s >= A[prev].f then
			X.append(A[i])
			prev = i
	return X
```

  

Whats the run time? → $\Theta(nlogn)$﻿. We need to be able to prove the correctness of this algorithm:

**To show correctness, will need to show (1) feasible and (2) optimal**

- Usually, we like to show **feasibility directly** and **optimality by contradiction**
    - Suppose solution $O$﻿ is better than $X$﻿
    - Show this necessarily leads to a contradiction
- **Two broad** strategies for deriving this contradiction:
    - **Greedy stays ahead:** show every choice in X is “at least as good” as the corresponding choice in $O$﻿
    - **Exchange:** show $O$﻿ can be improved replacing some choice in $O$﻿ with a choice in $X$