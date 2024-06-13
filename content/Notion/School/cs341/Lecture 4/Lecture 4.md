## The Selection Problem

**Input:** An array **A** containing **n distinct** integer values, and an integer **k** between 1 and n

**Output:** The k-th smallest integer in A

- **Minimum:** is a special case when k = 1
- **Median** is a special case when k = $\frac{n}{2}$﻿
- **Maximum** is a special case when k = n

  

### QuickSelect

- Choose a random pivot, say at index $i_y$﻿
- Restructure so that for all $j< i_y, A[j] < A[i_y] $﻿ and for all $j > i_y, A[j] > A[i_y]$﻿
- So, the $k$﻿th smallest element is either:
    - $A[i_y]$﻿ if $i == k$﻿
    - If $k < i_y$﻿, then $k$﻿th smallest element in $A_L$﻿
    - If $k > i_j$﻿, then $(k-i_j)$﻿th smallest in $A_R$﻿
        - **Recursive call into** **$A_L$**﻿ **or** **$A_R$**﻿

![[Screenshot_2023-10-02_at_10.05.27_PM.png]]

**⇒ Most Optimistic Analysis**

The most optimistic outcome is a word where we **always** recurse on $\frac{n}{2}$﻿ elements, giving us the recurrence relation: $T(n) = T(\frac{n}{2}) + \theta(n)$﻿, which by **Master Theorem**, yields **$\theta(n)$**﻿**.**

  

**⇒ Worst Case Analysis**

The pivot value always ends up being in the $i_y = 1$﻿ position, and consequently, always recursing on the right, giving us the recurrence relation: $T(n) = T(n-1) + \theta(n)$﻿ which has time complexity $\theta(n^2)$﻿

  

⇒ **Average Case Analysis**

We define a **good** pivot $y$﻿ as good if it is $y \in (\frac{n}{4}, \frac{3n}{4})$﻿, that is, in the middle 50% of elements. With this, we recurse on at most $\frac{3n}{4}$﻿ elements. Whats the probability of a given pivot being **good?** Let’s form a sketch for this proof:

- Probability of a good pivot is $\frac{1}{2}$﻿, hence, every 2 recursive calls, we expect one **good** pivot
- Problem size is reduced from $n$﻿ to $\frac{3n}{4}$﻿after linear time work
- This gives: $T(n) = T(\frac{3n}{4}) + \theta(n) \in \theta(n)$﻿

  

So, **average case** of `QuickmSelect` is $\theta(n)$﻿runtime, which relies on getting a **good pivot** within $\theta(1)$﻿. `Median-of-medians QucikSelect` (MOMQuickSelect) ensures a **good pivot** is found inn every recursive call.

  

### Median-of-medians QuickSelect

First, we group the input array into 5 groups. The below is a good illustration of this:

![[Screenshot_2023-10-02_at_10.47.12_PM.png]]

![[Screenshot_2023-10-02_at_10.59.56_PM.png]]

The above slides provide the intuition for MOM QuickSelect. Here is the pseudo code for it:

```C
MOMQuickSelect(k, n, A)
	// base case
	if n <= 14 then sort(A) and return A[k]

	// divide and conqier to find medians 
	r = (n-5) / 10
	medians[1...(2*r+1)] = new Array()
	for i = 1...(2*r+1)
		B[1...5] = A[(5*(i-1)+1)..(5*1)]
		sort(B)
		medians[i] = B[3]

	y = MOMQuickSelector(r+1, 2*r+1, medians)
	
	// divide and conquer to find rank k
	(AL, AR, iy) = Restructure(A, y)
	if k == iy then return y
	else if k < iy then return MOMQuickSelect(k, iy-1, AL)
	else /* k > iy */ then return MOMQuickSelect(k-iy, n-iy, AR)
```

⇒ In this we are first an array of medians called `medians`

⇒ Then, we find the median of that set of medians (doing this recursively, and returning that median value)

⇒ Then, with the median of medians, we have the median of the current problem, perform restructure on that

⇒ Then, we proceed as normal, recursing into $A_L$﻿ or $A_R$﻿ based on the $k$﻿ value we are looking for and the median

  

**Note: This is the more space efficient approach, hence why are first recursing to find the median of medians. An alternative way (easier to understand as well), is to create a separate function responsible for finding the median of medians.**

![[Screenshot_2023-10-03_at_12.00.00_AM.png]]

So, the subproblem size is $\le \frac{7n}{10}$﻿, making the 2 final recursive calls above have runtime of $T(\frac{7n}{10})$﻿.

⇒ With this, we can get 2 parts for our final recurrence relation:

- $T(n) \in O(n) + T(\frac{n}{5}) + T(\frac{7n}{10})$﻿ if $n \ge 15$﻿
- $T(n) \in O(1)$﻿ if $n \le 14$﻿

![[Screenshot_2023-10-03_at_12.05.53_AM.png]]

![[Screenshot_2023-10-03_at_12.06.01_AM.png]]