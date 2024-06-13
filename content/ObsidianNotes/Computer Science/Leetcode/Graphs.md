### Floyd’s Tortoise and Hare Cycle Detection Algorithm

This detection algorithm works well in Linked List and Array representations, where there are `n+1` elements in the array with each element is within the range `1,n` inclusive.

> [!info] Educative Answers - Trusted Answers to Developer Questions  
> Level up your coding skills.  
> [https://www.educative.io/answers/why-does-floyds-cycle-detection-algorithm-work](https://www.educative.io/answers/why-does-floyds-cycle-detection-algorithm-work)  

The above website provides a proof of correctness for this algorithm. The idea is, in looking for cycle in a linked list, we have 2 pointers, the `tortoise` and the `hare`. The `hare` moves twice as fast as the `tortoise`, so we would have something like:

```C++
hare = hare->next->next;
tortoise = tortoise->next;
```

If one of the pointers reaches a `nullptr`, then we know there is no cycle. Otherwise, they will at some point be on the same `ListNode` if there is a cycle. In some problems, such as the finding the duplicate value in array above, we need to find the start of the cycle, as that is the value that has duplicate values. To do so, we move one of the pointers to the beginning of the linked list. Then, move both pointers one node at a time, until they meet again. The node they meet on is the value that has a duplicate in the linked list. Here is the problem:

> [!info] Find the Duplicate Number - LeetCode  
> Can you solve this real interview question?  
> [https://leetcode.com/problems/find-the-duplicate-number/](https://leetcode.com/problems/find-the-duplicate-number/)  

This algorithm runs in $O(n)$﻿ and is constant space.

> [!info] Educative Answers - Trusted Answers to Developer Questions  
> Level up your coding skills.  
> [https://www.educative.io/answers/why-does-floyds-cycle-detection-algorithm-work](https://www.educative.io/answers/why-does-floyds-cycle-detection-algorithm-work)  

---

### Boyer-Moore Majority Voting Algorithm

> [!info] Boyer-Moore Majority Voting Algorithm - GeeksforGeeks  
> A Computer Science portal for geeks.  
> [https://www.geeksforgeeks.org/boyer-moore-majority-voting-algorithm/](https://www.geeksforgeeks.org/boyer-moore-majority-voting-algorithm/)  

This is a $O(n)$﻿ run-time with $O(1)$﻿ space algorithm for **finding the maximum occurring element in an array,** _where the max occurrence is defined as anything_ _$\gt \lfloor{\frac{n}{2}} \rfloor$_﻿_._