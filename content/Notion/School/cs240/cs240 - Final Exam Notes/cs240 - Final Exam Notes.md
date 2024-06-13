# Hashing

We know look at one of the most important topics, hashing. Specifically, we are implementing dictionaries with hashing.

**⇒ Direct Addressing**

This is a situation where this is a specific range of valid keys: $0 \le k \le M$﻿, in which we store **(k,v)** in an array, where **A[k] = v**. Essentially, direct addressing is the method of accessing an element in an array by using its index as an address. This can be fairly efficient:

- **search(k):** check if A[k] is empty
- **insert(k,v):** A[k] = v
- **delete(k):** A[k] = empty

  

Drawback to this is space can be wasted (if M is something like 10, but we only have 2 things), and the keys must be integers

  

**⇒ Hashing**

Hashing utilizes direct addressing. The assumption is that we have infinite keys (such as $U = \{0,1,...\}$﻿, but can be finite. Then, we have a hash function, that maps the key for a smaller, finite, set of values. In essence, a function$f: U \rightarrow \{0,1,...,M-1\}$﻿

- **f** is called the hash function
    - Typical ones may look like: **f(k) = k mod M**
- We store the results of the hashing function as a indexable key into the hash table, which can then be accessed using direct addressing.

  

Now, what if there are collisions? For example, we insert a **kvp**, but **h(k)** already exists in the hash table. One solution is **chaining**. The following visual should very clear on this:

![[Screenshot_2023-04-19_at_4.37.24_PM.png]]

![[Screenshot_2023-04-19_at_4.36.50_PM.png]]

**Key Points: (1)** We insert to the front of the list for the bucket of a particular index we are inserting into **(2)** We apply MTF after searching

  

**Load Factor:** **$\alpha = \frac {n}{m}$**﻿. This is important in analysis, and further down, deciding when to rehash. **n** is the number of items, and **m** is the size of the hash-table. We will let the average bucket size = 𝛼.

- **insert** has run time $\theta(1)$﻿
- search, delete have run time $\theta(1 + |T[h(k)]|)$﻿, which is one plus size of bucket

However, this does not necessarily mean that average cost of search and delete is $\theta(1 + \alpha)$﻿ → If all keys hash to the same slot, 𝛼 is still the same, but now, the average runtime is $\theta(n)$﻿. We can derive further with randomization.

  

**⇒ Randomization**

Assume that a hash-function is chosen randomly. We use the **Uniform Hashing Assumption: any possible hash-function is equally likely to be used. We can show that under this assumption:** **$P(h(k) = i) = \frac {1}{M}$**﻿, which means hash-values of any 2 keys are independent of each other.

- We can prove that the expected size of a bucket T[h(k)] is at most 1 + 𝛼. Hence, **search, delete ⇒** **$\theta(1 + \alpha)$**﻿ **expected runtime**

  

To improve efficiency, it makes sometimes to re-hash the hash-table. We want to maintain a load balance of around $\theta(1)$﻿, meaning \#items in hash-table is proportional to \#slots in the hash-table. When the load factor is too high, we get more collisions, which we want to reduce. The same thing applies for when the load factor is too small (wasteful of space).

- **Say we have constants** **$c_1, c_2$**﻿ **where** **$0 < c_1 < c_2$**﻿**. Whenever** **$\alpha < c_1$**﻿ **or** **$\alpha > c_2$**﻿**, then, we want to re-hash.**
    - To do so, choose a new M’ (roughly 2M). If load factor becomes too small, rehash with smaller M’
    - find a new random hash-function, mapping: $h': U \rightarrow \{0,1...,M-1\}$﻿
    - create new hash-table, of size M’
    - reinsert all KVP
    - update T ← T’ and h ← h’

![[Screenshot_2023-04-19_at_5.22.53_PM.png]]

---

### **Open Addressing**

While chaining works, it still wastes space, especially if we still have many keys hashed to the same value. Chaining does not solve the issue of vertical space waste. We now introduce open addressing, which has some solutions:

**Terminology:**

- **Prob Sequence:** search and insert follow a prob sequence of possible locations for key k → h(k,0), h(k,1), h(k,2), …
    - Basically, check every space until an empty one is found, and place the entry there

This is idea of open addressing.

  

**⇒ Linear Probing**

Simplest probing technique: If h(k) is occupied:

- h(k,0) = h(k)
- h(k,1) = h(k) + 1
- h(k,2) = h(k) + 2
- etc.

We assume a circular array, that is, if something is supposed to go the last slot in the array, but full, the next place to look is beginning of the array. Here is an example:

![[Screenshot_2023-04-19_at_6.09.11_PM.png]]

![[Screenshot_2023-04-19_at_6.09.21_PM.png]]

When searching, we use **the formula for linear probing is:** **$h(k,i) = (h(k) + i) modM$**﻿**.** If the position is occupied, but not the key we’re looking for, move onto the next i. We keep moving. If we end up going to an empty slot, then, it means the searched key was not found. Deletion is the same as searching. However, after we delete, need to mark that slot as **“deleted”**. This is because, if we delete it, and then search for the slot below it, that operation may fail as it was originally moved there, but now, the slot is empty, so search will return not found.

![[Screenshot_2023-04-19_at_6.21.18_PM.png]]

![[Screenshot_2023-04-19_at_6.21.27_PM.png]]

### Double Hashing

**Idea:** Open addressing with probe sequence

Double hashing aims to address some of the issues that are found with linear probing. It has the form: $h(k,i) = h(h_0(k) + i · h_1(k)) modM$﻿ for i = 0,1,…

- In the above form, we are adapting our hash function.
- $h_0(k)$﻿ is our old hash function.
- $h_1(k)$﻿ is a new hash function, combining them into one complete hash.
    - **Relatively prime to M for all keys k**
        - If this were not the case, then the prob sequence would not explore the entire hash table
        - hence, why it is preferable to choose M to be prime. Making M’ also prime ensures this.
- **Double hashing with a good secondary hash function does not result in the clustering found in linear probing**
- Search, insert, and delete, all work the same, but with this different prob sequence. (Linear probing is actually special case of double hashing, with h(k) = 1
    - However, we still mark as **deleted** for deleting in double-hashing. Still the same thing, probing, so need this.

  

The second hash function must be chosen well. As $h_1(k)$﻿ is often selected with modular hashing, having $h_2(k)$﻿ also be modular can lead to dependencies (we want these to be independent). So, we let it be $h_1(k) = \lfloor (M-1)(kA - \lfloor kA \rfloor) \rfloor + 1$﻿. We do the +1 at the end to prevent h(k) = 0.

- In this hash function, **A** is a constant. A good number for **A** is $\frac {\sqrt 5 - 1} {2}$﻿.

  

### Cuckoo Hashing

This is another hashing strategy. Cuckoo hashing is when we create another hash-table. Each of these hash-tables have a different hash function. Say these are $h_0(k)$﻿ and $h_1(k)$﻿. Then, an item with key k can **only** be at $T_0[h_0(k)]$﻿ or $T_1[h_1(k)]$﻿.

- Search and delete take $O(1)$﻿ time.

![[Screenshot_2023-04-19_at_9.04.50_PM.png]]

![[Screenshot_2023-04-19_at_9.05.03_PM.png]]

Whenever we are trying to insert, it is filled, we replace that key. Then, we hash it and insert into the next hash-table. We continue this process of inserting, kicking and re-inserting, until all items are placed, or **failure.**

- The biggest risk with cuckoo hashing, is a situation where we have an non-ending cycle of insertions and kick-outs. We can detect a dead-loop like this after too many attempts **(2n)**
- If fail, rehash with larger M and new hash functions.
- How can we search in **Cuckoo Hashing?** Just check both arrays at the hash-value. Same as delete.
- For cuckoo hashing, the 2 hash-tables do not need to be the same size.
- **Load Factor:** **$\alpha = \frac {n} {|T_0| + |T_1|}$**﻿

![[Screenshot_2023-04-19_at_9.27.18_PM.png]]

  

# **Range-Searching in Dictionaries for Points**

Data can come in more than on dimensions. Namely, we can work with multi-dimensional data. For this course’s purposes, we only concern ourselves with 2D. To be able to efficiently work with 2D data, we introduce new data structures.

### → Quadtrees

Consider if we have n data points. The idea of a quad tree is a construct a box area around the n points, of size $[0, 2^k)$﻿ x $[0,2^k)$﻿. This setup allows for easier insertion/deletion. We cab find an appropriate k by looking at the maximum and minimum values **(x,y)** values among the n points. A quad-tree splits the area into 4 each time, with each higher levels of the tree responsible for larger regions, and lower levels being responsible for smaller regions.

- The main idea is, for a quad-tree, we continue to divide the area into 4, as long as that space has > 1 point in it
- The root corresponds to the entire square area.

![[Screenshot_2023-04-17_at_8.25.50_PM.png]]

![[Screenshot_2023-04-17_at_8.26.05_PM.png]]

The convention is, in the tree, for a node, its children be in the order **NE, NW, SW, SE.** Some of the nodes in the right diagram do not have 4 children: We can choose to omit the empty subtrees (that region has not points in it).

**⇒ Convention: Points on split lines belong to region on the right (or top)**

Building the quad-tree is straightforward: recursively partition the area, S, into 4 until there is 1 or 0 points in a region.

**⇒ Searching**

Searching in a quad-tree is largely the same as doing so in binary trees and tries. Given the point we are searching for, we determine which quadrant it must be in, and go to that child node. If we reach a leaf node, there are 3 possibilities:

- Leaf has point we are looking for → found
- Lead has a different point → not found
- empty subtree → not found

  

**⇒ Insertion**

There are 2 cases to consider:

- If we insert in the square overall region, then we first search for the point. We will get to the node where it needs to be.
    - If there is a point there, change that node to a region node, and assign it 4 children. The node’s original point is now a children along with the inserted point
    - If there is no point there (empty subtree), simply insert the new point there by expanding the empty subtree into one node
- If we insert outside, there is no need to rebuild the tree. Find a new k that encompasses old area with new point. Then, make the old tree a child of the new tree (that has larger size to accommodate new point).

![[Screenshot_2023-04-17_at_9.05.23_PM.png]]

![[Screenshot_2023-04-17_at_9.05.08_PM.png]]

![[Screenshot_2023-04-17_at_9.05.35_PM.png]]

![[Screenshot_2023-04-17_at_9.05.47_PM.png]]

**⇒ Deletion**

This is also very straightforward. We search for the point we are deleting. There are 2 cases after:

- If we delete it, and the node’s parent still has 2 or more children, do nothing and leave as is
- If we delete it, and now the parent node only has one child, and that child is a leaf, delete the parent and move the child into their place
    - Cannot do this if the one child is not a leaf

  

**⇒ Spread factor:** **$\frac {L}{d_{min}}$**﻿**, where** **$L$**﻿ **= side length of R (the overall square area), and** **$d_{min}$**﻿ **is smallest distance between 2 points**

![[Screenshot_2023-04-17_at_9.39.32_PM.png]]

![[Screenshot_2023-04-17_at_9.39.15_PM.png]]

So, from the above analysis, we arrive that the complexity to build the initial tree is $$﻿ worst case.

  

**⇒ RangeSearch**

This is when we are searching for a list of items that fit the range conditions. We follow the following conditions:

If the child is blue (as in the top right image), we do depth-first, and finishing recursing on that child, before continuing to the next child node, $p_5$﻿.

![[Screenshot_2023-04-17_at_9.45.11_PM.png]]

![[Screenshot_2023-04-17_at_9.45.22_PM.png]]

![[Screenshot_2023-04-17_at_9.46.09_PM.png]]

![[Screenshot_2023-04-17_at_9.46.23_PM.png]]

- Running time is number of visited nodes + output size
- There is **NO** good bound on the number of nodes visited, highly depends on the inputed points
    - Worst case, $O(nh)$﻿, we will need to visit all nodes worst-case
        - This actually ends up being worse than exhaustive search (brute-force, checking if every point falls into the range), but faster in practice

**⇒ Quad-tree Summary**

- Easy to generalize to other dimensions
- Easy to compute and handle
- Space potentially wasteful, but good if points are well-distributed

---

### → KD-Trees

One of the major problems with quad-trees, is they can be wasteful of space as they can become very unbalanced. If we have 3 points (2 very close, 3rd very far), we will end up making a lot of divisions to build a quad-tree for this 3 points. Kd-trees seek to address this by:

- split the region into 2 smaller regions of equal number of points
- can be split either vertically or horizontally
- alternating vertical and horizontal splits gives best range search efficiency

  

We first get the set of points (no need to build surrounding square surface anymore). Then, split vertically or horizontally (by half number of points) with the initial split being vertical (x-split). Alternate split type. Keep doing this until each region only has one node inside of it. We build this recursively using depth-first search. After construction, it should look like this:

![[Screenshot_2023-04-17_at_10.10.30_PM.png]]

  

There are 2 ways we can split the points each time (by x and then by y):

1. sort list of x-coordinates and y-coordinates, so breaking into 2 regions is just floor of length
2. Use quick select

We want to end with partitions $S_{x < m}$﻿ and $S_{x \ge m}$﻿. Create left-subtree for $S_{x < m}$﻿ and right-subtree for $S_{x \ge m}$﻿. For each of the sub-trees, we now want to split on y (horizontal). Each node keeps track of the splitting line. As we have $S_{x \ge m}$﻿ and $S_{y \ge ysplit}$﻿, it follows that points on split lines (the value also acting as the “split”) belongs to the top/right side. The runtime for constructing kd-tree is:

- Sloppy Recurrence: $T^{exp}(n) = 2T^{exp}(\frac{n}{2}) + O(n)$﻿, which resolved to $O(nlogn)$﻿ expected time
- We can improve this to $$﻿ by pre-sorting the points beforehand.

![[Screenshot_2023-04-17_at_10.44.05_PM.png]]

**⇒ RangeSearch**

The method for range search is largely the same as what it was before. We take the region we’re looking for. If it intersects the node’s region, we go and check both of its children. If the node’s region (keeping in mind the lines/splits that may have occurred higher in the tree), we add all of the node’s eventual leaves into set.

![[Screenshot_2023-04-17_at_10.52.58_PM.png]]

We can prove that the range search complexity: $Q^x(n) ∈ O(n)$﻿. Taking the size of output into account, the final one is: $Q^x(n) ∈ O(s + \sqrt n)$﻿, where s is number of leaves

**⇒ Remember: If a tree has 2 children per node and has k leaves, then it has k-1 internal nodes**

To prove the range search complexity, we take the graph (with all points and all division lines drawn), and we clearly define the 4 lines that make up the search range. Then using we recursively derive the complexity.

**⇒ Summary:**

- Storage $O(n)$﻿
- Height: $O(logn)$﻿
- Construction time: $O(nlogn)$﻿
- Range Query Time: $O(s + n^{1- \frac{1}{d}})$﻿, where d stands for dimension (so for the above, dimension 2, resolves to $\sqrt x$﻿

---

### → Range Trees

Quad-trees and kd-trees are both intuitive and simple. But, both may be slow for range search. We look to use one-dimensional ways to solve this (using ideas from BST and AVL trees). Let’s take a look at how we would do a range search for binary trees. Again, the method we follow are similar to the range searches we have seen for quad-trees and kd-trees.

  

**⇒ Range Search**

- We start at the root node. It is marked blue if either on of its subtrees may intersect the range query (which is always the case for the root node). These are boundary nodes.
- We mark a node as red so that range search is not called on this node. Node is red when it is an outside node as its subtrees do not intersect the range query
- We mark a node as green to mark that it is an inside node. This means all the keys in the subtrees are in range, and so, everything in the subtrees is added.

Like usual, we are performing a depth-first search. We fully recurse on the left tree before recursing on the right tree. Once we are finished, here is what it can look like:

![[Screenshot_2023-04-18_at_9.51.25_AM.png]]

As this is a binary search tree, the keys are returned in sorted order, as they are visited in a sorted order (left to right). We can modify this to get **modified BST Range Search.** Now, we search for a path $x_1$﻿ and a path $x_2$﻿, where x1 is the left boundary and x2 is the right boundary. To get this path, we perform binary search for the values 28 and 43, one at a time.

- First time, for 28, and we mark every node we visit as a **boundary node**
- Same thing for 43

Green nodes are are inside nodes:

![[Screenshot_2023-04-18_at_10.07.23_AM.png]]

This modified version is not more efficient for BST range search, but will be efficient when we move to 2D range search in range trees. We know v is a topmost inside node if:

- v is not in p1 or p2
- v’s parent is in p1 or p2, but not both
- if parent is in p1, then v is a right-subtree
- if parent is in p2, then v is a left-subtree

  

Running time is found by:

- $O(logn)$﻿ to find p1
- $O(logn)$﻿ to find p2
- $O(1)$﻿ to check if boundary nodes are in the range → $O(logn)$﻿ of them, so total is $O(logn)$﻿
- $O(1)$﻿ to check topmost inside node → child of a boundary node → at most $O(logn)$﻿ topmost inside nodes so total time is $O(logn)$﻿
- Report all descendants of topmost inside node, at most s, the output size
- So, with the above, total running time is $O(s + logn)$﻿

  

**⇒ 2D Range Trees**

This idea is built on top of the BST ideas. Essentially, we will have BST of key-value pairs, where we are ordering by x-coordinate key. This makes a one-dimensional range-search by x-coordinate efficient. But, to do a 2D range-search, we will need to be able to compute range-search on the y-coordinates. To do so with a normal BST, we would need to first get the range-search based on x. Take the results, and then check every result to see if their y-coordinate is in range. This could be very inefficient. To address this, we introduce 2D range trees.

![[Screenshot_2023-04-18_at_10.48.51_AM.png]]

![[Screenshot_2023-04-18_at_10.49.13_AM.png]]

With this type of auxiliary structure in place, it makes range-search much more efficient. Say we are trying to do RangeSearch(T, x1,x2,y1,y2), where we are looking for points that are:

- x1 ≤ x ≤ x2
- y1 ≤ y ≤ y2

So, we first perform regular BST range search with the x coordinates, finding the boundary nodes and the topmost inside nodes. We do not go inside of the topmost-inside nodes subtrees. This action of finding the boundary nodes and top-most inside nodes is $O(logn)$﻿. Then, we check if boundary nodes are in valid x and y range, returning them if so. Then, for every topmost node **v**, search in its associated tree by y-coordinate, returning any node that is in that y-range.

  

**⇒ Space Analysis**

- Primary tree T uses $O(n)$﻿ space
- For each v, with auxiliary tree T(V), it uses $O(|T(v)|)$﻿ space
- So summing this up over all nodes in the graph: $\sum_{v∈T} |T(v)| = \sum_{v∈T} $﻿ # of ancestors $\le \sum_{v∈T} logn = n logn$﻿

  

**Dictionary Operations**

- **Search(x,y):** Do a search for x in the primary tree T
- **Insert(x,y):** First, do a insertion as you would for a normal BST with x. Then, once inserted, travel up the tree from inserted node’s parent to root, adding (x,y) to the auxiliary trees of each node along the way
- **Delete:** Analogous to insertion

These actions can leave the BST unbalanced. We don’t want to use AVL trees, as the rotations make deletion and insertion slow. In addition, we would need to rebuild auxiliary trees every rotation. We actually allow a certain imbalance in range trees, and rebuild only when that imbalance passes a certain threshold. Thankfully, we won’t look at how this is done.

**RangeSearch:** Finding boundary nodes + finding topmost nodes + range search for each topmost inside node

- Running time for each topmost inside node: $0(logn) $﻿ topmost inside nodes, with $S_v$﻿ being the number of items returned from auxiliary tree of v. So, running time is $O(S_v + logn)$﻿ for one search (same as the above normal BST analysis). This repeated over all $O(logn)$﻿ topmost inside nodes gives: $O(s + log^2n)$﻿

![[Screenshot_2023-04-20_at_10.22.49_AM.png]]

---

# String Matching

### → Karp-Rabin Algorithm

This method incorporates a hash-table to hash the guess and substrings of the text. We can always map strings to numbers, so we will look at this method with numbers rather than strings. Nonetheless, the concept stays the same. Consider the following:

- **P = 5 9 2 6 5**
- **T = 3 1 4 1 5 9 2 6 5 3 5**
- **Standard Hash Function: h(x) = x Mod 97**

So, we are looking for `h(59265) = 59265 mod 97 = 95`. We hash every same length substring from the text. If the hash values are different, move on to the next number. If they are the same, as then compare every digit to be sure. A good hash function will reduce the need for this greatly, by increasing the probability that same hash value means same string.

**→ Quick Mention: For historical reasons, hashes are called** ==**fingerprints**==

![[IMG_A031735872DC-1.jpeg]]

So, hashing the entire value, especially with a standard hash function, is an `O(m)` procedure. Doing this for every single length m substring would be extremely time consuming, and is worse than brute force. Luckily, we are working with numbers, which share m-1 digits. So, we can calculate the next fingerprint very easily, and in `O(1)` time. Consider:

- **P = 5 9 2 6 5**
- **T =** ==**4 1 5 9 2**== **6 5 3 5**

So, here how we do this:

1. First, hash first substring: h(41592) = 41592 mod 97 = 76
2. Second, compute 10000 mod 97 = 9
    1. We do this as **P** is length 5. This number corresponds to P’s size

Then, as `76 != 95`, we move onto the next substring (==**15926**==). We perform the following computations to obtain the next fingerprint:

![[IMG_7A6BCB9B4DEF-1.jpeg]]

**Conclusion:** This has expected run time `O(m+n)`, and worse-case run-time `O(mn)`, which is, however, extremely unlikely (would require each substring to have same hash function as the has-value of key - extremely unlikely and would be a terrible terrible hash function)

**→ In the above, m is the length of the string being hashes, and n is the length of the text being searched**

→ The mod value at random to be a prime from {2,…,$mn^2$﻿}

  

---

### → Knuth-Morris-Pratt Algorithm

This matching algorithm relies on using matching prefix’s and suffix’s to reduce the amount of comparisons that normally would be done in a brute-force search, and has a run-time of `O(m+n)`. This algorithm needs to construct a **FailureArray** to save what indexes to move to in the event of a mismatch of characters. The below shows the entire process:

![[module09_olga-29.jpg]]

So, here how it works:

- **We first, like brute force, keep track of 2 indexes: i and j**
- Then, we follow the 3 rules at the bottom.
- If the current character of the guess and the current character of the text match, we move forward in both and keep looking
- If they are not the same, theres 2 cases:
    - If we are at the first character of the guess, so **j = 0,** then, we simply leave j = 0, and move onto the next character in the text (i + 1)
    - Otherwise, if j > 0, then we only need to backtrack until we have a prefix of the guess. Sometimes, this could be to j =0, and others, we won’t need to return to the beginning. We repeat this until we find a match or read end of text. So, it is, in principle, similar to brute force, but we are now more intelligent as to where need to backtrack to.

  

Now, this all depends on the **FailureArray**, which also needs to be implemented. What is the failure array though? In concept, it is the length of the longest matching prefix/suffix pair for each substring **P[0…j]** of pattern **P.** To compute it for questions, we can either:

![[Screenshot_2023-04-16_at_10.39.30_AM.png]]

Which is fine for tests. The approach we can also take is the same thing, performing Knuth-Morris algorithm for **P** on each of its substrings **P[1,…,j].**

Now, we would go and check every single (m-1) substrings in **P[1…j]** and see what the longest **valid** suffix is, however, this would be a `O(n^3)` ! We need a much faster way of producing this. To do so, we make a key observation:

**→ Observation: After processing T, final value of j is longest valid suffix of T**

So, what we do is, for each of the m-1 substrings, we compare them with the string itself P:

![[module09_olga-37.jpg]]

![[module09_olga-42.jpg]]

The above shows what happens for `j=1` and `j=6`. We do this for all 1 ≤ j ≤ 6, hence constructing the **FailureArray** for us.

  

![[module09_olga-48.jpg]]

![[module09_olga-49.jpg]]

  

---

### → Boyer-Moore Algorithm

This is the fastest pattern matching in practice on English Text. To use this algorithm, we create 2 auxiliary data structures to assist us, with the later one being optional for this course:

1. **Bad Character Heuristic**
    1. Basically, when we encounter a mismatch of characters, we want to shift the pattern until the characters match.
2. **Good Suffix Heuristic**
    1. Basically, when we encounter a mismatch of characters, we want to maintain the current matches that we already have (this will be a suffix of the current place we are at, as we are reverse matching).
        
        ![[Screenshot_2023-04-08_at_3.42.23_PM.png]]
        

  

In this algorithm, we are performing reverse-searching. This means that we are trying to match characters first at m-1, then all the way to 0. Why do we do this? With this approach, **bad character heuristic** can rule out many shifts with reverse searching. To see more about this characteristic, we need to introduce some additional things first.

  

**⇒ Last-Occurrence Array**

We build the last occurrence array to use for **bad character heuristic.** This is a simple array that contains the index of the last occurrence of a character in the pattern. In the array, the index’s will be used to represent the 26 letters are integers (in actual implementation). For now, you can just think of them as “character” indexes. Here’s how one would look like:

![[Screenshot_2023-04-08_at_3.50.20_PM.png]]

How do we index in this algorithm? Mostly the same as KMP, where we let `i` be the position in the text being searched, and we let `j` be the position in the pattern. Check is performed by `T[i] = P[j]` and the current shift is `i - j`. So, what is the shifting formula? Consider the following:

![[Screenshot_2023-04-08_at_3.56.51_PM.png]]

![[Screenshot_2023-04-08_at_3.57.00_PM.png]]

So, we cannot always blindly apply the shifting formula, which is why we make it conditional. Therefore, the unified formula is as follows:

![[Screenshot_2023-04-08_at_3.57.57_PM.png]]

![[Screenshot_2023-04-08_at_3.58.08_PM.png]]

This is sufficient to create a **Boyer-Moore Algorithm.** However, the other component used in this algorithm is the **Good Suffix Heuristic.** What is this? This heuristic aims to maintain the current matched part, and shift so that this part of the text is still matched. Consider:

![[Screenshot_2023-04-08_at_4.00.12_PM.png]]

![[Screenshot_2023-04-08_at_4.00.21_PM.png]]

So, the above show this idea. We will not go into how this array is computed. So, we are left with the final algorithm:

**Boyer-Moore Without Good Suffix Heuristic Boyer-Moore With Good Suffix Heuristic**

![[Screenshot_2023-04-08_at_4.02.01_PM.png]]

![[Screenshot_2023-04-08_at_4.02.22_PM.png]]

**→ Summary:**

- Runs very well, even when only using **bad character heuristic**
- Worst case is `O(mn)` with **bad character heuristic only**, but in practice, much faster
- With **good suffix heuristic,** can ensure `O(n + m + size of alphabet)` run time
- The algorithm must choose at every mis-match, whether to follow the **bad character heuristic or the good suffix**

  

---

### Suffix Trees

Up until now, we have been looking for a pattern within a text, often pre-processing the pattern to improve the efficiency of searching. What if we are now searching for many patterns P within the same fixed text P? This approach focuses on pre-processing the text to make searching better. This all depends on a unique observation:

**→ P is a substring of T, if and only if P is a prefix of some suffix of T**

- We store all of T’s suffixes in a compressed Trie → this is called a suffix tree

  

Here are some examples:

![[Screenshot_2023-04-08_at_4.23.11_PM.png]]

![[Screenshot_2023-04-08_at_4.23.27_PM.png]]

So, how we search for a pattern within this tree? Simple, we just traverse the tree based on the current character in the pattern. The only difference is now our search needs to be able to stop at a non-leaf node (as the pattern may be a substring of a suffix). What is the run time?

- **Building**
    - This takes `O(size of alphabet * n^2)` time, but can be done faster (`O(size of alphabet * n)`), but we won’t learn it
- **Pattern Matching**
    - essentially search for compressed tries
        - some changes needed (search can end earlier, not at leaf node)
    - run time is:
        - `O(size of alphabet * m)` , assuming each node stores children in a linked list
        - `O(m)`, assuming each node stores children in an array
- **Summary**
    
    - Theoretically very good, but construction is slow or complicated
    - Rarely used in practice
    
      
    

---

### Suffix Arrays

Now, we like the idea of suffix trees, but they can be too slow or complicated. Suffix arrays aim to use the same idea, but to allow for slightly slower run time, but much simpler and easier to build. What we do is we sort all the possible suffixes into an array that will hold indexes.

![[Screenshot_2023-04-08_at_4.44.19_PM.png]]

  

![[Screenshot_2023-04-08_at_4.45.46_PM.png]]

![[Screenshot_2023-04-14_at_8.59.05_PM.png]]

When it is sorted, we then can perform binary search to locate either the pattern itself, or a suffix that contains the pattern.

![[Screenshot_2023-04-08_at_4.47.01_PM.png]]

![[Screenshot_2023-04-08_at_4.47.11_PM.png]]

![[Screenshot_2023-04-08_at_4.47.29_PM.png]]

**Note:** The runtimes for BM above are with the **Good Suffix Heuristic** in mind

---

# Compression

**Overview:** So, what is compression? Compression is the process of reducing the size of data in order to save space or reduce the time needed to transmit data. Here are some import terminology:

- **Source Text:** The original data, string **S** of characters from the **source alphabet** **$$**﻿
- **Coded Text:** The encoded data, string **C** of characters from the **coded alphabet** **$$**﻿
- **Encoding:** Algorithm mapping source text to coded text
- **Decoding:** Algorithm mapping coded text back to the original text

  

**⇒ For this course, the coded alphabet is usually binary:** **$$**﻿

![[Screenshot_2023-04-14_at_10.49.14_PM.png]]

![[Screenshot_2023-04-14_at_10.49.35_PM.png]]

The compression ratio represents the degree of compression achieved by a data compression algorithm. It is a measure of how much the size of the compressed data has been reduced compared to the size of the original data.

The compression ratio is typically expressed as a ratio of the size of the uncompressed data to the size of the compressed data. For example, if a file was originally 10 MB in size and the compressed version is 2 MB, the compression ratio would be 5:1 or 80%, indicating that the compressed data is 20% of the original size.

**⇒ For this course, we will focus on physical, lossless compression**

- Physical: Only know the bits in the data, not their meaning
- Lossless: Always decodes S exactly (the counterpart of this is lossy, where decoding is approximate)

  

In order to compress data, we need to encode it. There are 2 approaches we can consider:

1. **Fixed-length Code:** All codewords have the same length
2. **Variable-length Code:** All codewords may have different lengths (can optimize this to allow for frequent = shorter length)

  

Encoding and decoding are like the two sides of a bijection. We must be able to uniquely encode and decode. In other words, there must not be any ambiguity when encoding and decoding. This is one of the downfalls of something like Morse Code.

- So, we will only consider **prefix-free** codes E (codes, meaning the function that maps/encodes a string to another).
    - This means the encoding of some string **c** will not be the prefix of any other encoding for any other string **c’**

### The Basics

Now, before we jump into various compression algorithms, we first look at the basics, as all of the following algorithms are based upon the same fundamental concepts.

**⇒ Prefix Free: No code word is a prefix of any other code word. Helps remove ambiguity in decoding.**

**⇒ Encoding**

When encoding, basic way is to have an array, where character of **S**, is the array index, and the encoding of the character, is the value at that index.

![[Screenshot_2023-04-14_at_11.10.05_PM.png]]

![[Screenshot_2023-04-14_at_11.10.20_PM.png]]

In actuality, we don’t really need the array for encoding. With the trie alone, we can obtain the encoding of a character. To do so, we visit each leaf node, and traverse all the way until the root node, appending the node bit to the front of the string. The run time for this would be $O(|T| + |C|)$﻿.

  

Now that we’ve reviewed the basics, we begin looking at some compression algorithms.

---

### Huffman’s Algorithm - Huffman Codes

For this, we let $$﻿. For a given **S,** the best trie can be constructed with **Huffman’s tree algorithm.** The algorithm is based on the idea of assigning shorter codes to more frequently occurring characters and longer codes to less frequently occurring characters. How does it work?

1. First, we calculate the frequency (number of appearances) corresponding to each character in **S**
2. Put each character into its own trie, each trie has its own frequency (equal to the character’s frequency in the beginning)
3. Then, repeatedly merge the smallest 2 frequency tries into one. So, new root node, and one trie is left sub-tree and other is right-sub-tree. We adjust the encoding values (path from root to leaf, character), to take into account the new branch. We do this until we are left with one final trie.

  

- Consider: **Text = GREENENERGY,** **$\sum_C = \{G,R,E,N,Y\}$**﻿
- So, we have frequencies: **G:2, R:2, E:4, N:2, Y:1**
- Tree:
    
    ![[Screenshot_2023-04-14_at_11.36.13_PM.png]]
    

For such a process, we can calculate the **compression ratio:** **$\frac {|C|\lceil log|Σ_C| \rceil} {|S|\lceil log|Σ_S| \rceil} = $**﻿**.** However, these frequencies (referring to the text example in the image above) are not skewed enough to lead a good compression. Now, for merging tries, simply going through an array and looking for the minimum two would be very inefficient. We can store the tries in a **MinHeap** as **Key-Value Pairs: (key, value) = (trie weight, link to trie)**

- So, **deleteMin** is called 2 times, then merge the tries, and then **insert** the trie back into the **min-heap.** Repeat until size of the heap is 1

  

![[Screenshot_2023-04-14_at_11.42.59_PM.png]]

![[Screenshot_2023-04-14_at_11.43.09_PM.png]]

---

### Run-Length Encoding

Now, we’ve seen fixed-length encoding, where every character has the same length encoding. Now, this can be useful if **S** has long runs of the same characters: something like **111111110000000**. We can look to encode a large run of characters into a much smaller size encoding. **An example of this is RLE: Run-Length Encoding.** For this, $\sum_S = \sum_C = \{0,1\}$﻿, but can be extended to non-binary alphabets. For things like this, we can think of the encoding array as a dictionary. Now, this dictionary can vary between different encoding algorithms. For **RLE,** the dictionary is uniquely defined by the algorithm. So, there is no need to pass it to the decoder.

  

**The encoding idea is as follows:**

- Something like **00000 111 0000** encodes into **0 5 3 4**
    - This means, if **S** starts with 0, **C** starts with 0, same thing for 1
    - The next number represents the length of the first chain of 0’s. Then the next number, 3, represents the length of the next chain of 1’s. We alternate like this, so we always know if we encoding is about a sequence of 1’s or 0’s.

![[Screenshot_2023-04-15_at_11.24.52_AM.png]]

- The above shows how we encode. For an integer **k,** we have $\lfloor log k \rfloor$﻿ many 0’s padded at the front. Then, the binary value of **k** appended after that blocks of 0’s. If **k** was already binary form, we can just take **|k| - 1** to get the number of 0’s needed. This process is very easy to encode as well as decode.

![[Screenshot_2023-04-15_at_11.28.13_AM.png]]

![[Screenshot_2023-04-15_at_11.28.23_AM.png]]

  

**⇒ Now, we’ll look at the decoding process for RLE**

![[Screenshot_2023-04-15_at_11.30.55_AM.png]]

![[Screenshot_2023-04-15_at_11.31.26_AM.png]]

The decoding process is straight forward. The first bit in **C** tells us we are starting with a block of 0’s. Then, after the next bit, we read in a block of 0’s. This value + 1 is the number bits we read again, forming the actual value for **k,** which we obtain from the dictionary (or simply, we know the decimal equivalent of the binary . Then, repeat this until we have decoded the entire **C.**

  

- ⇒ **Runtime is** **$O(|S| + |C|) = O(|S|)$**﻿**, as the encoded string is almost always shorter. Even if longer, by at most constant factor.**
- **⇒ When the dictionary is uniquely defined by an algorithm, not influenced by the input (unlike adaptive ones, which then will need to be passed to decoder)**

---

### Lempel-Ziv-Welch

Now, this algorithm uses a dynamic dictionary. What does this mean? LZW aims to take advantage of frequently used substrings.

![[Screenshot_2023-04-15_at_11.53.03_AM.png]]

![[Screenshot_2023-04-15_at_11.53.12_AM.png]]

When we do encoding, we use a trie to do so.

![[Screenshot_2023-04-15_at_11.55.05_AM.png]]

![[Screenshot_2023-04-15_at_11.55.18_AM.png]]

For the decoding process, it is somewhat similar. We again will need a dictionary, but instead of passing it from the encoding process, we will rebuild it. We will need an initial dictionary before we begin. The encoding process is a $O(|S|)$﻿ running time algorithm.

![[Screenshot_2023-04-15_at_12.37.20_PM.png]]

  

![[Screenshot_2023-04-15_at_12.37.35_PM.png]]

So, this is how we decode. We read in the encoding value. If it is in the dictionary, we easily find the corresponding text. Then, we want to add a new substring to the dictionary, in particular, the past text, plus the first character of the current **S**. If the current encoding value is not found in the dictionary, we let $S = S_p + S_p[0$﻿]. Then, as usual, we will place $S_p + S[0]$﻿ into the dictionary. Now, this method of storing can be inefficient. We can store through substrings … of the substrings … 哈哈.

![[Screenshot_2023-04-15_at_12.44.11_PM.png]]

  

![[Screenshot_2023-04-15_at_12.44.32_PM.png]]

The running time of this **LZW Decoding Algorithm** is $O(|S|)$﻿. Here are pseudo code for encoding and decoding:

![[Screenshot_2023-04-15_at_12.46.32_PM.png]]

![[Screenshot_2023-04-15_at_12.46.46_PM.png]]

  

**⇒ Works badly if no repeated substrings: Dictionary keeps growing, but nothing is being used**

---

### Bzip2

Bzip2 is a compression algorithm that uses 2 transformations and 2 encoding processes.

![[Screenshot_2023-04-15_at_12.49.34_PM.png]]

**⇒ MTF Encoding**

This encoding does not perform any sort of compression. Instead, all it does is transform our data into a more favourable format for compression.

![[Screenshot_2023-04-15_at_1.31.56_PM.png]]

  

![[Screenshot_2023-04-15_at_1.32.40_PM.png]]

![[Screenshot_2023-04-15_at_1.32.50_PM.png]]

  

Through the nature of this, the decoding process is very straightforward.

![[Screenshot_2023-04-15_at_1.33.47_PM.png]]

![[Screenshot_2023-04-15_at_1.33.54_PM.png]]

---

### Burrows-Wheeler Transform

The purpose of this to transform text into a form that is favourable for MTF transformation. What does this mean? If the original text has frequently occurring substrings, we would want the transformed text to have many runs of the same character (hence getting a 000..00 sequence after transforming with MTF). So, how does this encoding work?

  

We first need to append a string end character to the string, we usually use **$.**

![[Screenshot_2023-04-15_at_1.48.17_PM.png]]

![[Screenshot_2023-04-15_at_1.58.24_PM.png]]

Now, we want to sort the cyclic shifts. We can do this in an efficient manner through **MSD-Radix Sort**, with $ being the lowest/smallest character. Now, we can encode:

**⇒ Encoding**

To encode, we simply

![[Screenshot_2023-04-15_at_2.00.20_PM.png]]

![[Screenshot_2023-04-15_at_2.00.43_PM.png]]

**⇒ Decoding**

We know that we want to find the list of first characters of the unsorted list of cyclic. So, we want to track where each of these shifts goes in the shift, so that we can track those after we have sorted, to find the original text. When can do this as follows:

- We know that the shift that has $ at the very end and the beginning of this is ‘a’, is the first row (row 0) of the unsorted list of shifts. Then, as it is cyclic shift, we know that ‘a’ must now be at the end. So, we look for the next shift in the list. We continue doing this until we have found all shifts (in the order that they were prior to sort) so that we can take the first character of the shift. But, there will be many rows that end with ‘a’, so how to decide which one to choose?
    - The row with $ at the end has **a** at the front. So, of the sorted rows that start with **a**, we want to get the relative position of the row we are interested in. Say, of all sorted rows starting with a, we are at **kth** position. So, for all rows sorted that have **a** at the end, we take the **kth** one.

  

![[Screenshot_2023-04-15_at_4.33.44_PM.png]]

![[Screenshot_2023-04-15_at_4.35.14_PM.png]]

To be able sort the key-value pairs, we need a stable sort (so, something like bucket sort): $O(n + |\sum_S |)$﻿

So, to be able to make this connection in shortest amount of time possible, we make the last column of of the sorted shifts array a key-value pair. Where, we will have: **(key, value) = (char, sorted shifts array index).** Now, we can get the correct row in constant time (by referencing the first character of the row, through the previous key-value pair).

  

![[Screenshot_2023-04-15_at_4.39.24_PM.png]]

![[Screenshot_2023-04-15_at_4.39.36_PM.png]]

  

We repeat this process until we have gone through all of the rows in the sorted shifts array.

![[Screenshot_2023-04-15_at_4.41.29_PM.png]]

![[Screenshot_2023-04-15_at_4.41.37_PM.png]]

---

## External Dictionaries

For binary search trees, it is able to support special search due to the keys being special. However, what if we want to support nodes having multiple keys?

### 2-4 Trees

Here, each node is either:

- **1 KVP and 2 sub-trees (possibly empty)**
- **2 KVP and 3 sub-trees (possibly empty)**
- **3 KVP and 4 sub-trees (possibly empty)**

In 2-4 trees, the empty trees must be on the same level (which by logic, is the last level). Similar to the order property in BST, there is one in 2-4 Trees:

![[Screenshot_2023-04-20_at_10.40.01_PM.png]]

For 2-4 trees, height is $O(logn)$﻿, which will be proven later.

  

**⇒ Search: search(k)**

For 2-4 trees, search is the same as it is for BST. As the intervals between keys represent different ranges. We first check if its a key in the current node. If not, we go check one of the appropriate sub-trees.

  

**⇒ Insertion: insert(k)**

This is a bit different then insertion in BST. First, we do search for k, leaving us at the node it would appear in, if it was in the tree. This is will leave us at a leaf node.

- Insert at the leaf node returned by the search
    - Make sure empty subtrees are in the last level, invalid if not the case
    - Make sure order properties maintained

  

Sometimes, the node we are trying to add our key to, is full. Inserting would result in overflow, so, we need to split the node to resolve overflow. We split into 2 parts (can be not equal, as can have nodes of 3 keys). Then, we take one key (the middle one) and move it into the parent. Overflow can spill into the parent. Again, split. If parent is the root node, we split and create a new node. Then, move the middle value into the new node, making it the new root. Keys and corresponding trees in old root are now left-subtree, same for the right side.

**⇒ Insert(17)**

![[Screenshot_2023-04-20_at_11.15.33_PM.png]]

![[Screenshot_2023-04-20_at_11.15.47_PM.png]]

![[Screenshot_2023-04-20_at_11.16.12_PM.png]]

![[Screenshot_2023-04-20_at_11.16.24_PM.png]]

  

Here are some quick definitions:

![[Screenshot_2023-04-20_at_11.18.28_PM.png]]

![[Screenshot_2023-04-20_at_11.18.38_PM.png]]

  

Now, deletion is where things get hectic. To see them, check the slides. Here are just the formal rules:

![[Screenshot_2023-04-21_at_12.11.06_AM.png]]

![[Screenshot_2023-04-21_at_12.15.47_AM.png]]

### (a,b) - Trees

We just looked at 2-4 trees. These are actually a type of (a,b) tree.

- (a,b) tree satisfies that each node has at least **a** subtrees, unless it is the root
    - root must have at least 2 subtrees
- each node has at most **b** subtrees
- if a node has **d** subtrees, then it will have **d-1** KVPs
- again, all empty subtrees are at the same level
- again, the subtrees represent the ranges between the keys in a node
- Also, a and b cannot b just be any 2 random integers chosen. The condition must be:
    - a ≤ $\lfloor (b+1)/2 \rfloor$﻿

  

Why does the root need to have at least 2? We do this so that trees with only 1 kvp are possible (if we made it ≥ 3, then there would need to be 2 kvp in the root node, as # kvp = # children - 1. As we have made the requirement that a ≤ $\lfloor (b+1)/2 \rfloor$﻿, insertion, deletion, and searching work exactly the same as for 2-4 trees.

- In particular, this condition makes overflow solvable. If we had the numbers to close, upon overflow, we would not be able to split the node into 2 valid nodes.
- Overflow means the node now has b+1 trees. So, we split this into $\lfloor (b+1)/2 \rfloor$﻿ and $\lceil (b+1)/2 \rceil$﻿, where this sums to b+1 always.
- Height is the same as all trees learned so far → number of levels excluding root and empty-tree row
- **Height of tree with n KVP is:** **$O(\frac {logn} {loga})$**﻿**.**
    - So, running time is $O(h)$﻿ which is $O(\frac {logn} {loga})$﻿

  

### B-Trees

B-trees are (a,b)-trees that are tailored to external memory. Each block in external memory stores one node of the tree. We choose a value for b, so that a node holds the largest node possible (b subtrees) fits into one block.

- So, each node has b-1 keys, b subtree references

An issue is that recall for (a,b)-trees, height is $O(\frac {logn} {loga})$﻿ which in turn determines the run-time of this data structure. However, if our a is a small value, we end up wasting a lot of space in memory and bad running time. So, we want to select the largest a possible for the chosen b value:

- So, we use a $(\lceil b/2 \rceil, b)$﻿-tree, where a is $\lceil b/2 \rceil$﻿.
    - this is called a B-Tree, where b is the **order** of the tree.

![[Screenshot_2023-04-21_at_9.47.23_AM.png]]

![[Screenshot_2023-04-21_at_9.49.24_AM.png]]

  

**Useful Fact for (a,b)-trees: number of of KVP = number of empty subtrees – 1**

![[Screenshot_2023-04-21_at_11.06.53_AM.png]]

So, for something like this, remember:

- # of nodes in B-Tree ≤ ${order}^{h}$﻿
- # of KVP in B-Tree = ${order}^{h+1} - 1$﻿
    - Number of empty subtrees is equal to ${order}^{h+1}$﻿ (aka, the last row)
- These are max, as under the assumption there are b sub-trees per node

  

**→ For a BST, the number of leaves is always one more than the number of internal nodes. So, for B-Trees, there are more empty trees in the last level, then internal nodes.**

So, there are ${order}^{h}$﻿ many leaves ≥ # of nodes in B-Tree. That is how we solve.