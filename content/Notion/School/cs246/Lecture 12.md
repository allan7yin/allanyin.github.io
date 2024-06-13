  

|   |   |
|---|---|
|`Copy Constructor`|`Copy Assignment Operator`|
|It is called when a new object is created from an existing object, as a copy of the existing object|This operator is called when an already initialized object is assigned a new value from another existing object.|
|It creates a separate memory block for the new object.|It does not create a separate memory block or new memory space.|
|It is an overloaded constructor.|It is a bitwise operator.|
|C++ compiler implicitly provides a copy constructor, if no copy constructor is defined in the class.|A bitwise copy gets created, if the Assignment operator is not overloaded.|
|Syntax: `className(const className &obj) { // body }`|Syntax: `className obj1, obj2; obj2 = obj1;`|

```C++
Student mahmoud{60,70,80};
Student daria{mahmoud}; // copy constructor 
Student alice; // defualt constructor 
alice = daria // copy, not construction - alice already exists, copy assignment operator
```

Classes come with copy assignment operator (CAO) the built-in behaviour just assigns all fields. Thus, you may need to write your own (e.g. built-in would shallow copy node).

```C++
struct Node {
	int data;
  Node *next;
  Node(int data, Node *next): data {data}, next {next} {} // constructor

  Node(const Node &n): data {n.data}, // copy constructor 
                       next {n.next ? new Node{*n.next} : nullptr} {}

	Node &operator=(const Node &other) { // copy asisgnment operator 
		data = other.data;
		delete next; // dangerous 
		next = other.next? new Node{*other.next}: nullptr;
		return this;
	}
}
```

**Why is the above dangerous?**

```C++
Node n{1, new Node {3, nullptr}}};
n = n;
```

When writing operator `=`, always consider self-assignment. The copy operator above is very wrong. We need to make a deep copy version:

**(Perhaps ask for more clarity on the below)**

```C++
Node &Node::operator=(const Node &other) {
	// we return node by reference because we want to be able to cascade equality
	// (a == b ==c)

	if (this == &other) return *this; 
	// make sure self copy does not destroy itself 

	data = other.data;
	Node *tmp = other.next ? new Node(*other.next) : nullptr;
	// we do this in case heap runs out of storage (safety)

	delete next; // important, free currently used memory if preivous step worked
	next = tmp;

	return *this;
	// dereference this and then return the reference 
}
```

The correct version above is a lot safer basically.

This link is a basically perfect explanation of the **Copy Swap Idiom:** [https://www.youtube.com/watch?v=7LxepUEcXA4](https://www.youtube.com/watch?v=7LxepUEcXA4)

  

So, we get this in the end:

```C++
\#include "node.h"
\#include <utility> //includes swap

//create special swap for node
void Node::swap(Node &other) { 
	std::swap( data, other.data );
	std::swap( next, other.next ); 
}

Node &Node::operator=(const Node &other) {
	Node tmp{ other }; //equivalent to tmp = other; deep copy happens here
//tmp is created on the runtime stack 
//tmp deep copied using copy constructor
	swap(tmp); //uses the swap we defined 
//move deep copy data into current node 
//tmp will go out of scope automatically and free this 
	return *this;
//self assignment is safe here, so we don't need the check necessarily
}
```

  

**Copy and Swap Idiom**

The copy and swap idiom is commonly used to implement an assignment operator that provides the strong exception guarantee for a resource managing class. Suppose we start the below class:

```C++
class DynArray {
	private:
		int m_size;
		int *m_data;
	
	public:
		DynArray(int size = 0) : 
			m_size{size}, m_data{m_size ? new int[m_size]{} : 0} {}

		DynArray(const DynArray &other) :
			m_size{other.m_size}, m_data{m_size ? new int[m_size] {} : 0} {
					std::copy(other.m_data, other.m_data + m_size, m_data);
			}

			~DynArray() {
				delete[] m_data;
			}
};
```

So, the above is the definition of the class `DynArray`. In the above, we need we’re missing our assignment operator. We know that a assignment operator will return a reference to itself:

```C++
DynArray &DynArray::operator=(const DynArray &other) {
	if (this != other) { // checks for self assignment 
		delete [] m_data;
		m_data = nullptr;

		m_size = other.m_size;
		m_data = m_size ? new int[m_size] : 0; 
		std::copy(other.m_data, other.m_data + m_size, m_data);
	}

return *this // as said before, returns a reference to object 
}
```

- Now, the above will work for most of the time. It doesn’t leak any memory. However, it does not maintain the state of an object if an exception/error is thrown.
- For example, say we go into the constructor and we first delete the data associated with the object through `delete [] m_data;` , rest the size using `m_size = other.m_size;`, and then, imagine that when we are “new”-ing an array, an error occurs and an exception is thrown. Well, while we aren’t leaking any memory, the state of our object has changed. We want to avoid this and provide a strong exception guarantee.
    - Strong exception guarantee -- **If the function throws an exception, the state of the program is rolled back to the state just before the function call.**
- We accomplish this through the copy and swap idiom.

  

First, we change the order of the above code. One way we can do this is to move the operations as follows:

```C++
DynArray &DynArray::operator=(const DynArray &other) {
	int size = other.m_size;
	int *data = m_size ? new int[size] : 0;  // now, if exception is thrown here
																					 // object state is not affected, "this" is still valid
		
	std::copy(other.m_data, other.m_data + m_size, m_data);

	delete [] m_data;
	m_data = data;
	m_size = size;
	return *this; // as said before, returns a reference to object 
}
```

Since the operations we are doing are solely on temporary objects, we no longer need to test for self assignment.

  

Now notice that the first three lines are literally doing what the copy constructor does. So, we can just instead create a temporary object to be a copy of the passed in parameter. Then, we need to swap the data of the copy and the object we want (`this`). So, we can use the `std::swap`. So, after these changes, our final code looks like:

```C++
DynArray &DynArray::operator=(const DynArray &other) {
	DynArray tmp{other};
	
	swap(*this, tmp);

	return *this; // as said before, returns a reference to object 
}
```

---

**Move Semantics**

The final part of this lecture is looking at move semantics.

Recall:

- an l-value is anything with an address

Now consider:

```C++
Node plusOne(Node n) {
	for (Node *p = &n; p; p = p->next) {
		++p->data;
	}

	return n;
}

Node n{1, new Node{2, new Node {3, nullptr}}};
Node n2 = plusOne(n); // this uses a copy constructor if elision is not used 
```

If the definition of n2 involves the copy constructor (it does), when what is other? Other itself is an l-value reference but what object does it refer to? The compiler creates a temporary object to be the return value of plusOne, and the other is the reference to this temporary, and the copy constructor deep copies this temporary.

  

So, an l-value and r-value are different types of values in c++. For example, look at the code below:

```C++
string firstname = "allan";
string lastname = "yin";

string fullname = firstname + lastname; 
```

So in the above code snipped, `firstname`, `lastname`, and `fullname` are l-values. That is, these variables have assignable memory addresses. On the other hand, the values `“allan”`, `“yin”`, and `firstname + lastname`, are r-values, temporary pieces of data that do not have accessible address. Say we have the function:

```C++
void printName(string &name) {
	cout << name << endl;
}
// a ver simple prit function that prints out a name given its reference

//so

printName(fullname); // this is fine since fullname is an l-value 
printName(firstname + lastname); // THIS IS WRONG - not a l-value
```

Now, note, if we were to modify the printName function as follows:

```C++
void printName(const string &name) {}
	cout << name << endl;
}

printName(fullname);
printName(firstname + lastname); 
```

Both print functions would be fine. So, if we write a function that expects a `const reference`, then passing it l-values or r-values will work.

  

However, a better and more clear way of doing so is to write a function that expects r-value references. We do so with `&&`

```C++
void printName(const string &&name) {}
	cout << name << endl;
}
// now, printName expects r-value reference

printName(fullname);
printName(firstname + lastname); // ok since firstname + lastname is r-value 
```

This is handy since now, we can just overload `printName`, and pass it either l-values or r-values, does not matter.

  

**Suppose we have the class below:**

```C++
class String {
	public:
		String() = default;
		String(const char *string) {
			std::cout << "Created!" << endl;
			m_size = string.size();
			m_data = new char[m_size];
			
```