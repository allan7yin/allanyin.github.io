  

## Module 1: Asymptotic Analysis

When talking about run-time analysis. We often want to analyze code. We associate such a cost with the number operations that are being done and count the number of primitive operations being done. We look at asymptotic analysis and growth rate to determine the time complexity of certain code. Namely, as the input size becomes ever larger → infinity.

**⇒ Order Notation: Big O**

![[Screenshot_2023-02-26_at_4.30.01_PM.png]]

![[Screenshot_2023-02-26_at_4.30.41_PM.png]]

These are some very simple and easy examples, but they get the point across. How about finding the run-time of code? Here is an example:

![[Screenshot_2023-02-26_at_4.32.57_PM.png]]

This on the left is example of how to analyze code:

- Count primitive operations, just say they are **constant time**
- So, we do loop n times, so n + c which is **O(n)**
- We apply this same sort of thinking for all types of code analysis, and is used again in calculating things such as expected rune time and average run time

  

Big O is nice, and often times, we may only care about the O notation of an algorithm as we care more about upper bound on running time. However, if we something like 2n is O(n^10), this is not really helpful. We now introduce a lower bound.

**⇒ Asymptotic Lower Bound**

![[Screenshot_2023-02-26_at_5.51.36_PM.png]]

This is lower bound, very similar to how we prob upper bound. Take note of the right side’s second proof. A tip is to always try and use a larger variable, like n^2, to remove a smaller one, like n, from the equation since the larger one will grow much larger eventually, and we can always just change the value of `n0`.

![[Screenshot_2023-02-26_at_5.52.10_PM.png]]

![[Screenshot_2023-02-26_at_5.55.12_PM.png]]

  

  

**⇒ Tight Asymptotic Bound**

This is just upper and lower bounds combined into one formal definition:

![[Screenshot_2023-02-26_at_5.58.20_PM.png]]

![[Screenshot_2023-02-26_at_5.58.38_PM.png]]

**⇒ Strictly Smaller and Strictly Greater**

![[Screenshot_2023-02-26_at_6.01.53_PM.png]]

![[Screenshot_2023-02-26_at_6.02.04_PM.png]]

**Another** way to proof run time, besides the above which is called first principles is using **limit theorem:**

![[Screenshot_2023-02-26_at_6.15.31_PM.png]]

![[Screenshot_2023-02-26_at_6.16.10_PM.png]]

![[Screenshot_2023-02-26_at_6.19.19_PM.png]]

  

![[Screenshot_2023-02-26_at_6.19.01_PM.png]]

The above are some small things to recognize and may be very helpful when solving/proving relationships between run-time.

- Some small notes to keep in mind
    - When proving things like O and Ω, just do the work and find the c and n0.
    - When proving things like o and w, then, after the rough work has been done (find n < something c, or n > something c), formalize a proof. Such as:

> After the preliminary work is done, the actual proof is as follows: Let c > 0 be a given. Let n0 = 125/c. Then for all n ≥ n0, n ≥ 125/c, so, 125 ≤ cn. So, we have 25n^4 + 100n^2 ≤ 25n^4 + 100n^4 = 125n^4 ≤ cn * n^4, = cn^5. This formally proves that 25n^4 + 100n^2 is of O(n^5)

**⇒ Algorithm Analysis**

Two tips:

- **Strategy 1:** Either use Θ bounds throughout the entire analysis and obtain the Θ for the complexity of the algorithm
- **Strategy 2:** Prove a O bound and a matching Ω bound separately, in order to arrive ay the Θ bound for the algorithm at the end
    - This can be potentially harder to deal with

**Strategy 1:** You just straight up sum, and from this, find the final running time complexity

**Strategy 2:** Sometimes, strategy 1 can be hard to directly solve for. So, use strategy 2. This method often involves some sort of manipulation of summation bounds, in order to create the desired inequality. Below, the left image shows upper bound, and the right image shows lower bound.

![[Screenshot_2023-02-26_at_6.31.48_PM.png]]

![[Screenshot_2023-02-26_at_6.32.02_PM.png]]

**To become more accustomed to these kinds of problems, its best to redo some of the A1 questions, as well as look at the tutorial ones.**

## Module 2: Priority Queues + Heaps

**⇒ Priority Queue ADT**

Collection of key value pairs, where the key is a priority. The most often, and common, operations used on priority queues are `**insert**` and `**deleteMax**`. There are max-heaps and min-heaps. Only different is the order in which the keys are maintained within the heap. Now, as this is an ADT, there are many ways to implement it. One of the best ways to implement this is through heaps:

**⇒ Binary Trees**

Before moving on, let’s get some review in on binary trees and their properties. Each level i, has 2^i nodes. So, for example, root node by itself is considered height 0, which means there is one node.

- **a tree with n nodes, of smallest possible height h, is:** `h ≥ log(n+1) -1`

There are some conditions that must be maintained to be considered a **max-heap**

- All parent keys > its children keys
- Items are left-justified. Meaning we add items to the left side and move right in a row
- Each row is filled (except possibly, the last)
- **Height of a heap with n nodes, is Θ(log 𝑛)**

  

Now, we can use an array to store the max-heap data structure, as using a linked list for heaps wastes space. When using an array format, we can access children with:

- Let A[i] be the current node
- **left child,** if it exists, is **A[2i +1]**, **right child**, if it exists, is **A[2i +2]**
- Parent of A[i], if exists, is **A[****_floor((i-1)/2)_****]**
- Can hide the implementation details using helper functions like parent(), left(), right(), etc.

  

**⇒ Insertion in max-heaps**

Do to this, we add the node to the end of the array, which is the first free leaf. Then, we perform **fix-up**, which swaps the node and its parent, if the child is now larger than the parent. Keep doing this up, until parent is larger than child. This is a **O(logn)** operation.

**⇒ DeleteMax in heaps**

To do this, we first swap the value of the root node, with the last leaf, since it is easier to delete the last node (as it is last element of array). Now, perform **fix-down** on the root node (which is now a small value), until heap order property is restored. This is a **O(logn)** operation.

  

**⇒ Sorting using heaps**

Sorting with heaps is fairly straightforward, and there are 2 ways of doing so:

1. The first way, is the dumb way. We construct a max heap. Then, insert each element of the array into the heap. Then, delete-max n times, and place each value in the array from back to front. Now, this is not very efficient, and not in place (we need an additional heap, of size n).
2. The second way to do this, is to sort within the array, and treat it as a heap. We take last node’s parent and call fix-down. Then, we go left in the array, and call fix-down for every single node until we reach the root node. This helps us construct a max-heap in place. We call this process **heapify()**. Has pseudo-code like:

```C++
heapify(A)
A: an array
for i <- (last()) downto 0 do
	fix-down(A, i)
```

Careful analysis reveals this to be **O(n).**

Now that we can construct a max-heap from an array in place, we can then uhmove onto to sorting it. To do this we:

1. **Heapify()** the array
2. **Swap** the root node, and the last node (last element in the array).
3. Decrease the size by 1 (effectively meaning, the last index of the array is now “out of bounds”, as it is already sorted)
4. Fix-down the root node
5. Repeat until the size of the heap is 1, and what remains is a fully sorted array

---

## Module 3: Expected Runtime, QuickSelect, QuickSort, Decision trees, MSD/LSD/Bucket Sort

⇒ **QuickSelect:**

- Worst case runtime is $O(n^2)$﻿, without random pivot
- Expected runtime with random pivot is $\theta(n)$﻿, generally the fastest implementation of a selection algorithm
    - Quick select uses **Hoare’s Partition** to first partition the elements into 2 sections, one less than the value used to partition, and the other greater than pivot.
    - We can take the last index value, A[n-1], as the pivot value. Have pointer to the first element, **i,** and pointer to the second last element, **j**. They move towards, each other. When A[i] > pivot and A[j] < pivot, we swap A[i] and A[j]. We stop when **i** and **j** cross each other. Then, we replace A[i] with A[n-1]. Now, A[0…j] is the partition that is less than pivot, and A[i…n-1] is the partition that is greater than pivot
    - This is a $O(n)$﻿ operation.
- Quick select continually searches for the value. If it is less than pivot value, recurse into left partition. Otherwise, recurse into the right partition.

  

**In general, expected runtime is the same as expected runtime.**

  

⇒ **QuickSort:**

Sorting that uses **Hoare’s Partition**. First, we partition into the 2 halves. The right partition is greater than the left partition, so, we have “kind of” sorted in 2 bundles. We recurse onto those 2 partitions. After recursing, array is now sorted.

- **Worst Case:** **$O(n^2)$**﻿
- **Best Case:** **$O(nlogn)$**﻿
- With a random pivot:
    - **Expected Running Time:** **$O(nlogn)$**﻿
    - **Average Running Time:** **$O(nlogn)$**﻿

  

**⇒ Comparison Trees:**

This is a way we look at that justifies why for comparison based sorting algorithms, $O(nlogn)$﻿ is the best we will ever do.

**→ Comparison:** No restriction on input, just need to be able to compare

**→ Non-comparison:** Restrictions on input, but able to achieve $O(n)$﻿ sorting

  

What do these do? Consider the comparison tree for 3 non-repeating elements. In that case, all such instances are of the form [x0, x1, x2]. Now, these can be in any sorted order. Now, the tree would look like this:

![[Screenshot_2023-04-21_at_7.05.48_PM.png]]

So, as seen above, for sorting 3 elements, the minimum # of comparisons needed is 2 (left path, for example). So, on the MT, answer is not 5log5. Instead, need to picture this kind of tree for 5 elements, and figure out the shortest height. It is from this, we can then prove that the **any comparison-based sorting algorithm requires at least** **$\Omega(nlogn) $**﻿**.**

**⇒ REMEMBER: Binary tree with height h, has at most** **$2^h$**﻿ **leaves. This is later applied to B-Trees (but instead of 2 children, there are b many)**

  

**→ For a BST, the number of leaves is always one more than the number of internal nodes. So, for B-Trees, there are more empty trees in the last level, then internal nodes.**

  

**⇒ Non-Comparison-Based Sorting**

**⇒ Bucket Sort**

We have a fixed range (0,…,L-1], for example. We sort into buckets that are contained at each index, adding multiple to one bucket if there are multiple instances of that value. This is $O(L + n)$﻿. Something important to note about bucket sort, is that it is indeed stable.

- We can do a bucket sort on last digit of numbers. First, we assume all numbers are length M (pad 0’s in front if not the case). We sort the numbers, digit by digit.

  

![[Screenshot_2023-04-21_at_8.43.18_PM.png]]

So, we sort by digits, as seen above. This leads into MSD and LSD radix sort. Something we want to use is **Base R number representation.** That is, assume (or convert) numbers into a certain base. This way, we know there are at most R different digits. This makes it possible to sort by digits.

- **Runtime:** **$\theta(n + R)$**﻿
- **Auxiliary Space:** **$\theta(n + R)$**﻿, where n is for linked list, and R is for the buckets

  

**⇒ MSD-Radix-Sort**

Not much needs to be said. First, sort into groups based on the most significant digit (most left one). Then, within each group, sort by the second digit. We keep recursing like this until we obtain individual number nodes:

![[Screenshot_2023-04-21_at_8.48.27_PM.png]]

So, for MSD-Radix:

- Auxiliary space is: $\theta(n + R + m)$﻿
    - The space for bucket sort + recursion depth (m-1)
- Total run time: $\theta(mnR)$﻿

  

**⇒ LSD-Radix Sort**

Now, we start from the least significant digit. Now, this is stable sorting algorithm. We just do bucket sort m times, for each digit position, from m-1 to 0.

![[Screenshot_2023-04-21_at_8.54.05_PM.png]]

  

**Runtime:** **$\theta(m(n+R))$**﻿

**Auxiliary Space:** **$\theta(n + R)$**﻿

---

## Module 4: BST, AVL Trees

- **Binary Search tree**
    - How to insert, search, and delete
    - For BST, insertion and deletion are `O(h)`, where best case `h = logn`, and worst case `h = n`
- **AVL Trees**
    - Balance factor = height(v.right) - height(v.left)
        - +1 means right heavy, -1 means left heavy, 0 means balanced
    - **AVL Tree on n nodes as Θ(log 𝑛) height**
        - `N(h) = N(h-1) + N(h-2) + 1 >= N(h-2) + N(h-2) = 2N(h-2)`
        - This is recurrence relation for minimum number of nodes for height h for AVL trees
    - After insertion or deletion, may need to restore AVL tree to maintain **height-balance property**
        
        - There 4 cases, adjust using rotations:
            
            ![[Screenshot_2023-02-25_at_8.43.16_PM.png]]
            
        - **Case 1:** Right rotate on node z
        - **Case 2:** Left rotate on node z
        - **Case 3:** Left rotate on y, then right rotate on z, **double right**
        - **Case 4:** Right rotate on y, left rotate on z, **double left**
        - Here is the pseudo-code:
        
        ![[Screenshot_2023-02-25_at_8.56.51_PM.png]]
        
    - Can solve all 4 cases **separately**, using the 4 rotation methods, or just using **tri-node** restructuring:
        - Middle node becomes new root node. Largest is right child, smallest is left child. Then, right child of middle becomes left child of largest node. Left child of middle becomes right child of the smallest node.
    - Here is pseudo-code for insertion, remember to update height of each node from inserted node up to root.
        
        ![[Screenshot_2023-02-25_at_8.49.00_PM.png]]
        
    - **Deletion**
        - Deleting from AVL is straightforward. First, find node want to delete, and then, delete it normally as you would with deleting from a BST. Then, update height, and check balance property for all nodes from deleted node’s parent, to root node. Whenever unbalanced, fix it.
        - In the diagram below, node z is realized to be unbalanced. When calling restructure, we pass to it node z (the unbalanced one), its tallest child, and its tallest grand child. As both grandchildren are the same height, we need to choose one of them. So, if y is left child, we pick y’s left child, otherwise, if y is right child, we pick y’s right child. In other words, we always create either case 1 or 2 unbalanced situation, so that only 1 rotation is required to balance.
            
            ![[Screenshot_2023-02-25_at_8.52.54_PM.png]]
            
        - Here is he pseudo-code for deletion, and module 4 summary:
            
            ![[Screenshot_2023-02-25_at_9.09.06_PM.png]]
            

---

## Module 5: Skip Lists + Ordering Heuristics

==**⇒ NOTE: Height of skip list is different from height of a tower. Height of tower is the number of nodes above the one in s0 (so 0 or more). Height of the skip list is literally seeing the maximum number of nodes that are within one of the towers.**==

- **Skip-Lists:** are the randomization of binary search trees as a data structure and another realization of the dictionary ADT. Skip lists aim to mimic binary search with a **hierarchy** of linked lists. Each item has a **tower**, in which we flip a coin as to whether it grows or not (hence the randomization factor). Will look something like this:
    
    ![[Screenshot_2023-02-25_at_9.30.38_PM.png]]
    
    - **Searching:** This is done through getting through getting the nodes needed to get to the desired node. In skip-lists, whether it be for searching, deletion, or insertion, we always end up at the node directly before where what we are looking for would be (sometimes does not exist). Here is some pseudo-code:
    
    ![[Screenshot_2023-02-25_at_9.27.05_PM.png]]
    
    - **Insertion:** This is largely the same as search. We get the predecessors for the key we want to add, taking us directly before where we need to insert. Then, we add it, adjusting the references (as we are working within a linked-list). Next, flip-coin, until we get a tails. The number of heads, is height of tower for the newly inserted node. The image above shows insertion. Here is the pseudo-code for it:
        
        ![[Screenshot_2023-02-25_at_9.31.20_PM.png]]
        
    - We maintain a stack of nodes, which is what `getPredecessors(k)` does, and returns it. From this stack, we pop the top element, and insert k after that node. We randomize a value i in the beginning (flipping coin), and if the tower is larger than the current height of the skip-list, the skip-list needs additional layers to be added above. We add a node for each layer of the tower, hence the while `i > 0` loop condition.
    - **Deletion:** Again, similar to insertion. We `getPredecessors(K)` to find where the node is. Then, we delete it from every S0 layer. At the end, if there are multiple empty sentinel only lists, reduce height until there is only one. Here is the pseudo-code:
        
        ![[Screenshot_2023-02-25_at_9.36.25_PM.png]]
        
    - **Skip-list Analysis:** So, now let’s do some analysis on skip-lists. Below are the slides that do so.

  

![[Screenshot_2023-02-25_at_9.48.03_PM.png]]

![[Screenshot_2023-02-25_at_9.48.20_PM.png]]

![[Screenshot_2023-02-25_at_9.48.33_PM.png]]

![[Screenshot_2023-02-25_at_9.48.48_PM.png]]

- In the above, highlighted in blue are the key points. Also some useful, **very useful**, things to remember, like sum from 0 to infinity of 1/2^i is 2.
    - **Expected Space:** This, for skip lists, is Θ(𝑛). For 2 reasons:
        - **Space for sentinels** = 2h + 2, for obvious reasons
        - **Space for storing keys** is at most `E[2h + 2] = 2E[h] + 2 ≤ 2(2 + logn) + 2 = 6 + 2logn`
    - **Expected Running Time:** This is combination of dropping down, and scanning forward. We will drop-down h times, so **expected height h is O(log n)**, as shown above. **Expected number of scan-forwards is also O(log n)**, so overall **expected running time** is **O(log n).** Let’s look at how we derived **O(log n)** for the scan ahead portion:
        - We look at for the ith level.
            - Assume i < h, since if we are on level h, there will be no scanning forward as that row only has the sentinels.
            - Let `v` be the leftmost key in `Si`, we visit during search as reached `v` from `Si+1` . Let `w` be the key right after `v`. So, what is the probability of scanning from `v` to `w`?
                - If we do scan forward in `Si` from `v` to `w`, this means `w` does not exist in `Si+1`, otherwise, it would have went there
                - So, scanning to `w` means that tower of w has height of i
                - We scan forward from `v` to `w` with probability of at **most 1/2,** as we could go down if key < `w`.
                - So, the probability of scanning forward l times, is **(1/2)^l**
                    
                    ![[Screenshot_2023-02-25_at_11.59.44_PM.png]]
                    
                - Here is the final slide, showing how we get **O(log n)**
                    
                    ![[Screenshot_2023-02-26_at_12.00.18_AM.png]]
                    
    - As the way skip-lists are implemented above, they are really no faster than **randomized binary search trees** (These are BST, but where a new node is inserted is randomized.
    - We can improve skip-list implementation by making towers arrays. In this implementation, each tower is an array, and above the `S0` node, the array holds references, which then point to the next array tower. This improvement aims to save space by eliminating some other references.
        
        ![[Screenshot_2023-02-26_at_10.32.40_AM.png]]
        
- **Skip-list: Summary**
    - For skip-list with n items, expected space usage is **O(n)**, and expected runtime for search, insert, and delete, is **O(log n)**
    - **2 efficiency improvements:**
        - Use arrays as towers (as mentioned above)
        - **Can show:** a biased coin-flip to determine tower height gives smaller expected run times
        - With these 2 improvements, skip-lists are **fast** in practice and relatively easy to implement
- **Optimal Ordering of Items**
    - We want to make search in unordered arrays/lists more effective. We can accomplish this **if the items are not accessed in equal probability**
        - 2 cases for this: **(1)** We know the distribution beforehand, and **(2)** we do not know beforehand
    - For static (know before) ordering, very straightforward:
        
        ![[Screenshot_2023-02-26_at_10.41.05_AM.png]]
        
    - For dynamic ordering, there are 2 techniques we can employ. These are MTF heuristic, and Transpose Heuristic. Two important terms to know are **online** and **offline algorithms.** An **online** one is an algorithm that processes input data as it become available. So, for here, MTF and Transpose are online algorithms. On the other hand, offline algorithms are ones that receive all of its input data at once, so it knows beforehand (static ordering belongs here).
        - **MTF Heuristic:** Move the most recent accessed item to the front of the list (easy for LL, but for array, will probably need to search from the end, in order to move things to the “front” of the array).
            - MTF is “2-competitive”, which means that no more than twice as bad as the optimal offline ordering. In this case, it means that MTF is no more than twice as bad as optimal static reordering.
            - One downside of MTF, is that sometimes, the changes can be very drastic. If a not frequently accessed element is called, then it is now at the front. This then increases access cost for many keys before it ends up at the end again. To improve this, you could consider instead using Transpose.
        - **Transpose Heuristic:** When search for an item, move it up by1, instead of moving to the very front of the list.
    - Worst case, **MTF and Transpose** run Θ(𝑛) (as worse case they are always fetching the item at the end of the list).

---

## Module 6: Search + Interpolation Search + Tries/Trie Variations

- Is there a lower bound for search? In a comparison based model, Ω(log 𝑛) are required for search. We can prove this with a decision tree. Consider we are searching from n items. So, the decision tree will have `n+1` nodes (n if we find the node, and +1 for not found option). We also know that for binary trees, for height h, there are at most 2^h leaves. So, 2^h ≥ n + 1, so h ≥ log(n+1).
- We can beat this lower bound if the keys are special, however. We introduce **interpolation search**, a variation of binary search. These two work in the same idea:
    
    - Have left index and right index, and find middle. If k is less than A[middle], then recurse on left sub-array, otherwise, recurse in the right sub-array. In the case of binary search, we split the array in half every time. But, what if we could find it faster by not dividing by 1/2? If our array is uniformly distributed, or close to, it may faster to use interpolation search. This is the same idea as binary search, but instead of splitting in half each time, we divide by how close k is t the right or left side, as seen below:
    
    ![[Screenshot_2023-02-26_at_11.35.15_AM.png]]
    
    - When recursing, the array we are recursing into has expected size **root(n)**. The recurrence relation is:
    
    $Tavg(n) = Tavg(\sqrt{n}) + Θ(1) ∈ Θ(log logn)$
    
    - The worse-case performance of interpolation search is Θ(n). It is easy to imagine when. For instance this happens when have multiple instances of k, or, if we have a very large variation, something like 1,2,3,4,5,..9,10,999999. Then, each time we calculate the new “middle” index, it will be very small. So, we end up iterating through the entire array.
- **Tries:** These are dictionaries for bit strings (01). We can compare these with AVL trees to see more of why they are used:
    - Let `x` be a string, so `|x|` is the length of that string. In tries, search, insert, and delete strings are all O(|x|) time. They are independent of the input size. Now, what about AVL trees?
        - AVL trees would require `O(|x|logn)` time, as we have `O(logn)` for search, insert, and delete. But then, when comparing keys, as we are not dealing with numbers, we need to compare character by character, which is O(|x|) run time.
    - A standard trie has two options per node: 0 or 1. However this works only for prefix-free. To relax this, we also allow $ as a child for a node. So although a “binary trie”, there can be 3 children.
        
        ![[Screenshot_2023-02-26_at_12.11.05_PM.png]]
        
    - When searching in tries, we just read in each character along some label. When inserting, we just first search. Then, where we cannot go any further (as no matching child), we insert the remaining nodes there.
        
        ![[Screenshot_2023-02-26_at_12.12.48_PM.png]]
        
    - When deleting, we first search for the item. Once we reach the leaf node, we go up, deleting the node if it has no other child. **Time complexity of insert and delete is Θ(|𝑥|).**
- Those were standard tries. There are also many variations, which all aim to reduce the amount of space used by tries.
    - **Variation 1: No Leaf Labels**
        - Instead of actually storing the word at the leaf, we just implicitly store the word as the entire branch from root to bottom. This reduces a lot of space.
            
            ![[Screenshot_2023-02-26_at_12.56.10_PM.png]]
            
    - **Variation 2: Allow Proper Prefixes**
        - This one is a “proper” binary tree. Only two options: left means 0, right means 1. Each node can be marked wth a flag, to indicate if it is a word or not:
            
            ![[Screenshot_2023-02-26_at_12.57.51_PM.png]]
            
    - **Variation 3: Removing chains to leafs**
        - Sometimes, we have long chains, single nodes strung together, towards a leaf node. Instead of having the entire chain, we can just only show the final leaf node:
            
            ![[Screenshot_2023-02-26_at_12.59.07_PM.png]]
            
- **Compressed Tries**
    - Here, we store a value (index) at each node. The value tells us how many beginning characters match, and so, can skip over them when searching. A **compressed tree with n keys has at most n-1 internal nodes. So, compressed tries have at most 2n-1 nodes.**
        
        ![[Screenshot_2023-02-26_at_1.01.16_PM.png]]
        
    - Say we search the compressed tree above for the key 110$. So, read in 1, so go right. Then, the node as 2, which tells us every before and including the second character, are the same. So, we can skip over the second 1. Now, we go down the path with label 0. We then see the 3, meaning every before and including index 3 are the same. So, all have 0 as third character. Then read in $, and go left. Found. Here is the pseudo-code for search:
        
        ![[Screenshot_2023-02-26_at_1.56.19_PM.png]]
        
    - **Search and Delete for Compressed Trees**
        - These operations are both very easy to implement conceptually
        - For deletion, we first search. Then, we remove the node where we have the string, and compress. In the above example, imagine that I want to delete the string 110$. So, first search and we arrive at that leaf node. Then, we delete it. Now, if the the parent node now only has one we also delete the parent, and attach the other child to the grand parent. So, in the example above, we delete 110$. Then, delete the node with value 3, and move 1101$ as the new left child of the node with value 2.
- **Multiway Tries**
    - Now, above we only really looked at binary strings. Consider if we had alphabet other than only 0,1.
        
        ![[Screenshot_2023-02-26_at_2.26.38_PM.png]]
        
    - Notice that the run-time complexity of search, insert, and delete is `O(|x| * (time to find appropriate child))` as instead of there only be 2 ways to branch from a node, there is an entire alphabet of options, which we can implement through a number of ways listed above.