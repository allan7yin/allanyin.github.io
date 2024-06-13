### Complexity Class NP-Complete

→ The complexity class **NPC** denotes the set of all decision problems $\Pi$﻿ that satisfy the following 2 properties:

- $\Pi \in$﻿ **NP**
- For all $\Pi' \in $﻿ **NP ,** **$\Pi' \le_P \Pi$**﻿

**NPC** is an abbreviation for **NP-Complete**

> Note: The definition does not imply that NP-Complete problems exist

  

**→ Satisfiability and the Cook-Levin Theorem**

![[Screenshot_2023-12-03_at_4.43.17_PM.png]]

This is the above theorem says that the above **CNF-Satisfiability** is NP-Complete, where we always abbreviate **CNF-Satisfiability** as **SAT.** This result has some huge implications, and it was the first algorithm to be proven to be **NP-Complete.** If you think about it, this theorem proves that for all **NP** problems in the past, present, and future, they will all be polynomially reducible to **SAT** (pretty wild to think about tbh).

  

We can take this known NP-Complete SAT, and via polynomial reductions, prove that new problems are also NP-Complete (_basically, if we can use polynomial reductions to convert the instances of SAT into those of ours, then we can run the Oracle on that modified instance and return its return value_). Let’s see how to prove that **new problems are also NP-Complete**

  

**→ Proving Problems NP-Complete**

Given an NP-Complete problem, say $\Pi_1$﻿, other problems in **NP** can be proven to be **NP-Complete** via polynomial transformations from $\Pi_1$﻿, as stated in the following theorem:

**Theorem:** _Suppose that the following conditions are satisfied:_

- $\Pi_1 \in$﻿ NPC
- $\Pi_1 \le_P \Pi_2$﻿, and
- $\Pi_2 \in$﻿ NP

Then $\Pi_2 \in$﻿ NPC

---

In the remainder of the lecture, we will learn how to prove that **3-SAT (a different problem that represents a special case of the SAT problem) is also NP-Complete.** We’ll also take a look at **2-SAT** which turns out is **not NP-Complete** and is intact solvable in polynomial time.

![[Screenshot_2023-12-03_at_5.17.38_PM.png]]

**→ Now is SAT Harder than 3-SAT? Only polynomially**

Prove **SAT** **$\le_P$**﻿ **3SAT**

- Suppose $(X,C)$﻿ is an instance of **SAT**, where $X = \{x_1, ..., x_n\}$﻿ and $C = \{C_1, ..., C_m\}$﻿
    - $X$﻿ is the name of all variables → assume for simplicity, they are 1,2,…n → don’t really need this when we make this assumption
    - $C$﻿ is a list of clauses → each clause contains some literals
- For each $C_j$﻿, do the following:

![[Screenshot_2023-12-03_at_6.36.04_PM.png]]

Essentially, for different length clauses, we find equivalent 3-literal wide logical equivalents. The one to pay close attention to is the last case. This above should build some **strong intuition as to why SAT** **$\le_P$**﻿ **3SAT.** We now formally prove this:

  

**→ Correctness**

- Want to prove: **SAT** **$\le_P$**﻿ **3SAT**
- i.e our transformation function satisfies:
    
    1. $f(I)$﻿ is computable in poly-time, for all $I \in \mathcal{J}(\Pi_1)$﻿
    2. If $I \in \mathcal{J}_{yes}(SAT)$﻿ then $f(I) \in \mathcal{J}_{yes}(3SAT)$﻿
    3. If $f(I) \in \mathcal{J}_{yes}(3SAT)$﻿ then $I \in \mathcal{J}_{yes}(SAT)$﻿
    
      
    

**(1)**

We’ll just provide a quick sketch of this idea. Let $L$﻿ be the number of literals in input $I$﻿. In our transformation input, we construct at most $4L$﻿ **clauses.** Clearly. this can be done in $poly(4L)$﻿, which is in $poly(L)$﻿, which is $poly(Size(I))$﻿

  

**(2) If** **$I \in \mathcal{J}_{yes}(SAT)$**﻿ **then** **$f(I) \in \mathcal{J}_{yes}(3SAT)$**﻿

- Since **$I \in \mathcal{J}_{yes}(SAT)$**﻿**,** then there is some satisfying truth assignment, we fix $X$﻿ to be this assignment
- We consider each clause $C_j$﻿ of the instance $I$﻿
    - If $C_j = \{z\}$﻿, then $z$﻿ must be true. The corresponding four clauses in $f(I)$﻿ each contain $z$﻿, so they are all satisfied
        - $\{z,a,b\}, \{z,\bar a,b\}, \{z,a,\bar b\}, \{z,\bar a,\bar b\}$﻿
    - If $C_j = \{z_1, z_2\}$﻿, then at least one of $z_1$﻿ or $z_2$﻿ is true. The corresponding 2 clauses in $f(I)$﻿ each contain $z_1,z_2$﻿, so they are both satisfied
        - $\{z_1, z_2, c\}, \{z_1, z_2, \bar c\}$﻿
    - If $C_j = \{z_1, z_2, z_3\}$﻿, then $C_j$﻿ occurs unchanged in $f(I)$﻿
    - Suppose $C_j = \{z_1, z_2, z_3, ..., z_k \}$﻿ where $k \gt 3$﻿ and suppose $z_t \in C_j$﻿ is a true literal. Define $d_i = true$﻿ for $1 \le u \le t-2$﻿ and define $d_i = false$﻿ for $t-1 \le i \le k$﻿
        - We can intuitively understand why this is. By making $d_i = true$﻿ for all $1 \le i \le t-2$﻿, we are essentially making all the previous clauses true. So, why can’t we do this moving forward? We note that the very last clause is of the form $\{\bar{d_{k-3}}, z_{k-1}, z_k\}$﻿. Now, we don’t know what the $z$﻿ values are, but to make this true without them, we would need $d_{k-3} =false$﻿. So, every clause after the one with $z_t$﻿ will have their $d_i$﻿ value false.
        - This means every clause will have at least one thing making them true (one with $z_t$﻿ guaranteed to have 2)

  

**(3) If** **$f(I) \in \mathcal{J}_{yes}(3SAT)$**﻿ **then** **$I \in \mathcal{J}_{yes}(SAT)$**﻿

We’re going to **consider each clause** **$C$**﻿ **in the SAT input** **$I$**﻿**.** We identify a corresponding set $S$﻿ of clauses in $f(I)$﻿, and we **show** **$C$**﻿ **must be satisfied** because of the clauses in $S$﻿

![[Screenshot_2023-12-03_at_8.01.04_PM.png]]

![[Screenshot_2023-12-03_at_8.01.15_PM.png]]

Last step is showing **3SAT** **$\in NP$**﻿

We’ll review this, recall the steps to follow are:

- Define data type of certificates
    - Basic data type: int, string, array of ints, … (nothing fancy things like sets and maps)
- Defined desired **YES-Certificate** format
- Design a poly-time `verify(I,C)` algorithm
- Correctness proof:
    - **Case 1:** Let $I$﻿ be any yes-instance → find $C$﻿ such that `verify(I,C) = true`
    - **Case 2:** If `verify(I,C` is true → prove $I$﻿ is a yes-instance

  

So, let’s apply this to **3SAT:**

- **3SAT** input $I$﻿: a list of clauses, which are a list of literals, each literal is a tuple, containing a variable $\in \{1..n\}$﻿ & a negation bit
    - so, the **certificate data type** = array of bits (1 represents true for that variable, 0 represents false for that variable)
    - **YES-Certificate is an array** **$C$**﻿ **with one bit per variable representing a satisfying assignment**
- Next, we define a poly-time `verify(I,C)` algorithm:
    
    ![[Screenshot_2023-12-04_at_10.49.07_AM.png]]
    
    - The above takes $O(|Clauses|)$﻿ → polynomial in the input size
- Now, we prove correctness:
    - **Case 1:**
        - Let $I$﻿ be any yes-instance of 3SAT. That means, it has a satisfying assignment $A_S$﻿. And `verify(I,A_s)` will see that each clause contains a literal satisfied by this assignment, so `verify` will see that `numSat = |Clauses|` and return true
    - **Case 2:**
        - Suppose `Verify` is true, that means `numSat = |Clauses|`. Sop, `numSat` was incremented every iteration of loop of clauses, so each clause contains a satisfied literal → so, 3SAT formula in $I$﻿ is satisfied by $C$﻿ → yes-instance

  

**→ Thus, it follows 3SAT is NP. Since we have shown that SAT** **$\le_P$**﻿ **3SAT, we now know that 3SAT is NP-Complete**

![[Screenshot_2023-12-04_at_11.06.41_AM.png]]

(2 slides on 2sat, look at if you have time).