### Set Data Structure

To construct this, have the following options:

```C++
std:set<int> mySet;
mySet.insert(10);
mySet.erase(10);

// constructing set from an array with iterator 
std:vector<int> nums;
std:set<int> mySet(nums.begin(), nums.end());
```

  

### Map Data Structure

  

### Stack

There is a stack data structure in C++:

```C++
std:stack<int> st;
st.push(3);
st.pop();
st.top();
```

But, in my experience, it is just easier to use a vector. We have `push` and `pop` functionality with vectors. But also, if needed, we can iterate through the stack:

```C++
std:vector<int> stack;
stack.push_back(3); // push
stack.pop_back(); // pop
stack.back(); // top

for (int i = 0; i < stack.size(); i++) {}

// Stack can be implemented with vector
```