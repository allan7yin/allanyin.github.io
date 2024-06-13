`PrefixSum` is a common question that can be pretty challenging. However, I've noticed some common patterns to these types of questions. So, how can we approach these problems?

Upon first glance, we may think the answer lies in using `sliding window` techniques. However, often in these types of questions, moving the window will prevent us from finding the answer. For questions like this, the solution lies in storing information about the prefixes of each subarray, starting from the first index, into a data structure like a `hashmap` for instance.

The fundamental example of this type of questions is the **Subarray Sum Equals K** problem:

> [!info] Subarray Sum Equals K - LeetCode  
> Can you solve this real interview question?  
> [https://leetcode.com/problems/subarray-sum-equals-k/description/](https://leetcode.com/problems/subarray-sum-equals-k/description/)  

Consider the following screenshot:

![[Screenshot_2024-05-08_at_2.55.49_PM.png]]

What is this doing? Say our input array was: `nums = [1,-1,1,1,1,1]`, with `k=3`being our target sum. We initialize our hash-map with `0:1` since we can imagine a `0` value in-front of the array. For each number, we add it to a sum. Then, we do `sum - k`. Why?

- We know from `i = 0` to `i = current`, we have recorded `sum`. But, we want to get a subarray that sums to `k`. So, we need to **subtract** something from the beginning of the array. How do we know what we can subtract? Thats right, **we need to check if** `**sum - k**` **is in the hash-map**. This will tell us how many of such prefixes there are, and thus, how many subarrays of sum `k`, we can have up until and including, the current index.
- We continue this until we reach the end of the input `vector`

Hopefully this clarifies how to approach this problem. This is like the first `prefix-sum` problem that teaches us to solve these kinds of problems. Keep this technique in mind. Here is the final code for this question:

```C
class Solution {
public:
  int subarraySum(vector<int> &nums, int k) {
    unordered_map<int, int> prefix_sum_count;
    prefix_sum_count[0] = 1;

    int sum = 0;
    int count = 0;
    for (int &num : nums) {
      sum += num;
      int distance = sum - k;

      if (prefix_sum_count.find(distance) != prefix_sum_count.end()) {
        count += prefix_sum_count[distance];
      }
      prefix_sum_count[sum]++;
    }
    return count;
  }
};
```