# Sets

This is a data structure which holds unique keys, and is unordered. So, accessing elements of an unordered set is very efficient. There are **NO** duplicates. All operations on the unordered_set take constant time O(1) on average and linear O(n) in the worse cases. Here is the code for a set:

```C++
int main()
{
    // declaring set for storing string data-type
    unordered_set <string> stringSet ;
  
    // inserting various string, same string will be stored
    // once in set
  
    stringSet.insert("code") ;
    stringSet.insert("in") ;
    stringSet.insert("c++") ;
    stringSet.insert("is") ;
    stringSet.insert("fast") ;
  
    string key = "slow" ;
  
    //  find returns end iterator if key is not found,
    //  else it returns iterator to that key
  
    if (stringSet.find(key) == stringSet.end())
        cout << key << " not found" << endl << endl ;
    else
        cout << "Found " << key << endl << endl ;
  
    key = "c++";
    if (stringSet.find(key) == stringSet.end())
        cout << key << " not found\n" ;
    else
        cout << "Found " << key << endl ;
  
    // now iterating over whole set and printing its
    // content
    cout << "\nAll elements : ";
    unordered_set<string> :: iterator itr;
    for (auto itr = stringSet.begin(); itr != stringSet.end(); itr++)
        cout << (*itr) << endl;
}
```

  

The API is the same for unordered_set, its just that the elements inside of the set will not be ordered.

**⇒ Use Cases**

set is especially useful when we are dealing with duplication, and specifically removing duplicates. This works well since elements of a set are unique. When choosing to use set, choose between a set, or an unordered set. If the data we are working with is not needed to be sorted, than we should use an unordered set, as that results in more time efficiency.

**⇒ When to Avoid**

If we need key-value pair, should look to work with map, or if we need to access elements directly through index, a vector/array is a much better option.

**⇒ Note**

In addition, adding elements to a set have a time complexity of **O(logn)**

# Map

Each **key** is **unique.**

Now, in nature, maps and sets are fairly similar and serve similar purposes. They are both implemented via a self-balancing binary search tree. Maps are associative containers, containing key, value pairs. The key is used to uniquely identify the element and the mapped value is the content associated with the key. Both key and value can be any pre-defined type or user-defined type. Maps are very useful. Here is a short example of their use case:

```C++
int main()
{
    // Declaring umap to be of <string, int> type
    // key will be of string type and mapped value will
    // be of int type
    unordered_map<string, int> umap;
  
    // inserting values by using [] operator
    umap["GeeksforGeeks"] = 10;
    umap["Practice"] = 20;
    umap["Contribute"] = 30;
  
    // Traversing an unordered map
    for (auto x : umap)
      cout << x.first << " " << x.second << endl;
  
}
```

# Maps vs Sets - Breakdown

The difference is set is used to store only keys while map is used to store key value pairs. For example consider in the problem of **printing sorted distinct elements**, we use set as there is value needed for a key. While if we change the problem to print frequencies of distinct sorted elements, we use map. We need map to store array values as key and frequencies as value.

# Queue

The queue data structure is very simple. It is a first in first out data structure, and is useful in many applications. In particular, reference the “Map of highest peak” code to see how we used it. Please review when you have the time, as it was very **important.**

# Hashmaps/Hashtables

These are the same as the maps and sets. One thing to note, that for some things, the map will be able to hash for you. However, if we have, say a `map<int,vector<int>>`, the program will need to know how we are mapping a `vector<int>` into an integer. This function will need to be written by you.

# Priority Queue

Priority queues are a type of container elements, specifically designed such that the first element of the queue is either the greatest or the smallest of all the elements in the queue and elements are in non-increasing order (hence we can see that each element of the queue has a priority {fixed order}). So, essentially, if we push a bunch of integers into a priority queue, it’s no longer a FIFO structure. Instead, the front of the queue is the largest number, and it is sorted. For example:

```C++
int main()
{
    priority_queue<int> gquiz;
    gquiz.push(10);
    gquiz.push(30);
    gquiz.push(20);
    gquiz.push(5);
    gquiz.push(1);
  
    cout << "The priority queue gquiz is : ";
    showpq(gquiz);
  
    cout << "\ngquiz.size() : " << gquiz.size();
    cout << "\ngquiz.top() : " << gquiz.top();
  
    cout << "\ngquiz.pop() : ";
    gquiz.pop();
    showpq(gquiz);
  
    return 0;
}
```

This would result in the output:

```C++
// The priority queue gquiz is :     1    5    10    20    30

// gquiz.size() : 5
// gquiz.top() : 1
// gquiz.pop() :     5    10    20    30
```