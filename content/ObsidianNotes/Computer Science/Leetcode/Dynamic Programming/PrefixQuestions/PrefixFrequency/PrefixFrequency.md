# Example 1: Longest Substring With Even Vowels

This type of questions is very similar to the `PrefixSum` which is another folder in this directory. For reference, here is the question link: [https://leetcode.com/problems/find-the-longest-substring-containing-vowels-in-even-counts/description/](https://leetcode.com/problems/find-the-longest-substring-containing-vowels-in-even-counts/description/)

Now, what is the thought process behind this? Consider the following:

![[Screenshot_2024-05-08_at_2.53.59_PM.png]]

Instead of keeping prefix sums as in the `PrefixSum` problems, we are keeping a prefix frequency map, shown on the right side. Instead of maintaining a moving sum as we iterate through the string, we make use of a `bitmask`. Why? In this question, we don't care about the count of each vowel, rather, we only care whether the count is even or odd, something that can be represented in a binary format.

Recall that in `PrefixSum` questions, what we are trying to do, is essentially keep track of the moving sum, and look for a prefix that we can cu t off, to obtain a subarray of sum `k`. The same is true here. Instead of looking for a particular prefix sum, we are checking whether the current bit-mask has been encountered before.

Say the current bit-mask is `10101`, which means:

- `a` has appeared **odd** number of times
- `e` has appeared **even** number of times
- `i` has appeared **odd** number of times
- `o` has appeared **even** number of times
- `u` has appeared **odd** number of times  
    So, what would it mean to have seen this parity before? It means there prefix to this string that also has odd  
    `a`, even `e`, etc.

Why do we care? Well, if a prefix string has odd number of `a`, and the string up to the current index has odd `a`, then the middle (cutting off the prefix string) will give us a string with **even** `a` count: odd - odd = even. The same as is true for `e`, etc. -> even - even = even.

So, as you can see, this follows much of the same intuition as prefix sum questions. Here is the final code for this question:

```C++
class Solution {
public:
    int findTheLongestSubstring(string s) {
      int parity = 0;
      int max_length = 0;
      unordered_map<int, int> prefix_freq {{0:-1}};
      unordered_map<char, int> dp {
        {'a', 4},
        {'e', 3},
        {'i', 2},
        {'o', 1},
        {'u', 0}
      };

      for (int i = 0; i < s.size(); i++) {
        if (dp.find(s[i]) != dp.end()) {
          // current character is vowel, adjust bitmap
          int shift_pos = dp[s[i]];
          parity ^= 1<<shift_pos;
        }

        if (prefix_freq.find(parity) != prefix_freq.end()) {
          max_length = max(max_length, i - prefix_freq[parity]);
        } else {
          prefix_freq[parity] = i;
        }
      }
      return max_length;
    }
};
```

- Note the use of `parity ^= 1<<shift_pos;`
    - `^` is the XOR operator in most languages, for bit-wise operations
    - `1<<shift_pos` is shifting `1` left. e.g is 1 is `00001`, and `shift_pos = 3`, then this operation would return `01000`.
    - We use this to invert one particular bit in the bit-mask
        - e.g say we wanted to invert the first bit in `10101`. So, we first do: `1<<4 = 00001 << 4 = 10000`
        - `10000 XOR 10101 = 00101`

# Example 2: Number of Wonderful Strings

This question is very similar to the above. Here is the link: [https://leetcode.com/problems/number-of-wonderful-substrings/description/](https://leetcode.com/problems/number-of-wonderful-substrings/description/)

Here is the code for this question:

```C++
class Solution {
public:
    long long wonderfulSubstrings(string word) {
      // only characters from 'a' to 'j'
      int parity = 0;
      long long count = 0;
      unordered_map<int, int> prefix_count {{0,1}};

      for (int i = 0; i < word.size(); i++) {
        int shift = word[i] - 'a';
        parity ^= 1<<shift;

        // this is for 0 odd-number occurrences
        count += prefix_count[parity];

        // this is for 1 odd-number occurrence
        for (int j = 0; j < 10; j++) {
          int copy = parity ^ (1<<j);
          count += prefix_count[copy];
        }

        prefix_count[parity]++;
      }
      return count;
    }
};
```

So, the idea is the exact same. Now, we are looking to find the count of substrings that have **at most one character that has** odd count. Again, we will maintain a parity bit-mask and a hash-map of prefixes to help us with this.

Again, we compute the bit-mask on every character, and then, we check if we've already seen this parity before. This is because if we have, then cutting that prefix off creates a substring where every character appears `even` number of times: `odd - odd = even, even - even = even`.

But what about instances where we have **exactly one character of odd count?**. We handle cases like this in the `for (int j = 0; j < 10; j++) {` loop. Inside this, we increase the frequency of every character in every iteration, allowing for one more count of each character. This means, e.g say our current bit-mask is something like: `00000` `(a,b,c,d,e)`. So, every character has even count. Of course, if we see this as a prefix, we can increment our counter.

So, what if we've seen `10000`? Well we in this case, we have `even a's` but we see we've had a `odd a's` prefix. Removing that prefix would result in a `even - odd = odd` number of `a`. But this is ok, as long as there is only one, hence why we only increment count for each character once in the for loop (10 times since only characters from `a` to `j` are possible in this question). If the different is odd for only one character, we still want to count this, hence why we need this for loop.

The overall idea is very very similar to the top question. Remember this strategy.