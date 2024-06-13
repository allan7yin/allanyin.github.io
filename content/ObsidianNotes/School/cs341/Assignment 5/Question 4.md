### Part 1:

We prove that this problem is NP-Hard via by showing there exists a poly-time Turing reduction from the known **NP-Complete 3SAT.** The input is boolean formula $F$﻿ in $n$﻿ boolean variables, such that $F$﻿ is the conjunction of $m$﻿ clauses, where each clause is the disjunction of exactly three literals. We want to find a truth assignment to each of the $n$﻿ boolean variables that would make the formula $F$﻿ also be true. So, we can make the following comparisons:

- Each conjunctive clause represents a patron, as in the $\text{Chef-Alain’s-challenge}$﻿, we are looking for a dish assignment such that every patron is satisfied
- In the $\text{Chef-Alain’s-challenge}$﻿ problem, each patron can give up to 5 reviews of _“absolutely love”_ and _“absolutely dislike”_. We note that this is an upper bound, and so, letting every patron give exactly 3 reviews of that type works well, as we imagine each customer simply do not use their last 2 reviews. For 3 dishes in the list of variables, each patron/clause will give a truth assignment, where we correspond _“absolutely love”_ = to a boolean assignment of say $p$﻿ and _“absolutely dislike”_ = a boolean assignment of say $\neg{p}$﻿. For simplicity of analysis, we assume that $\text{Chef-Alain’s-challenge}$﻿ solves the problem of dish assignment in constant time, where it returns a list of dishes and each dish is assigned a boolean value, indicating whether it was chosen to be in the menu or not. So, we construct the following pseudo-code:

```Python
def 3SAT-Solver(X,C):
	# X is name of all variables -> we let each one correspond to a dish 
	# C is list of all clauses -> we let each one correspond to a patron

	menu = ChefAlainChallengeSolver(dishes=X, preferences=C)
	return menu
```

**→ Proof of Correctness**

Now, we argue that this reduction is correct. The algorithm takes a set of literals $X$﻿ and a list of clauses $C$﻿ as input, where each $c \in C$﻿ is comprised of 3 literals, where the literals can be negated versions of themselves. We recognize that this is the same as $\text{Chef-Alain’s-challenge}$﻿ problem. The `ChefAlainChallengeSolver` computes a truth assignment for the dishes to be on the menu. Each patron must be satisfied, similar to how each clause must be satisfied in **3SAT**. Each patron can give up to 5 reviews, but for this reduction, we consider the instance of when each patron gives exactly 3 reviews, which is perfectly fine input for the `ChefAlainChallengeSolver`. The algorithm finds a suitable assignment of dishes that satisfies every patron and returns this assignment. If such an assignment is not possible, where every patron is satisfied, then there exists an unsatisfied patron (the dishes they liked are not on menu and the dishes they hated are on the menu). In this case, `ChefAlainChallengeSolver` returns “impossible”. We return the menu directly, as it represents a truth assignment to the all variables such that each clause evaluates to true. If `ChefAlainChallengeSolver` returns “impossible”, this in turn means there is no truth assignment to variables in $X$﻿ that satisfy every clause in $C$﻿. Hence, this is a correct reduction. To justify it is a **polynomial** reduction, we show that the run-time on the input-size is polynomial.

**→ Runtime**

The input size is $|X| + |C| = n + m$﻿. We know for a reduction like this, we are make the assumption that `ChefAlainChallengeSolver` runs in constant time. Thus, it is evident that `3SAT-Solver` runs in poly-time on the input size. Therefore, we have shown a polynomial reduction $\text{3SAT} \le_P^T \text{Chef-Alain’s-challenge}$﻿, where we know $\text{3SAT}$﻿ is NP-Complete, we conclude that $\text{Chef-Alain’s-challenge}$﻿ is therefore NP-Hard.