```C++
//in lectures/c++/06-classes/04-rvalue/node.cc
\#include <iostream>

Node plusOne(Node n) {
	for ( Node *p{&n}; p ; p = p->next ) {
	++p->data;
	}
	return n; 
}

int main() {
	Node n {1, new Node {2, nullptr}};
	Node n2{plusOne(n)};
}
```

Assume that for the above, we have a regular constructor and a deep copy constructor. In the above, we use the regular constructor when me write `new Node {2, nullptr}`, and also when we make `Node n{1, new Node {2, nullptr}};`.

  

When we pass `n` to `plusOne`, we make a copy, using the copy constructor. Likewise, when we `return n`, we also call the copy constructor. Then, when we are initializing n2 with the return of `plusOne(n)`, we again, need to call the copy constructor.

  

So, in the above code snippet, we have a total of 8 constructor calls, 2 for regular and 6 for copy.

  

So, we can physically count 8 total uses of constructors. However, if we were to run a program to count this, the value may be smaller. This is because of `copy/move elision`. Elision refers to a compiler optimization technique that eliminates unnecessary copying of objects.

  

To compile a program **without** this compiler feature, we can:

```Bash
g++ -std=c++14 -fno-elide-constructors node.cc -o node 
```

```C++
struct Node {
	int data;
	Node *next;
	Node (int data,  Node *next) : data{data}, next{next} {}
	
	~Node () {delete next;}
};

int main() {
	Node n1{1, new Node{2, nullptr}};
	Node n2{3, nullptr};
	Node n3{4, &n2};
}
//when I leave main, destructor is automatically called 
```

==_****check understanding of this**_==

The problem here is that n2 is a stack allocated object, and once we need to delete n3, once we exit main, compiler will attempt to destroy n2 since n3 has a pointer to it. However, since it is a stack allocated object, will lead to undefined behaviour if we try to call destructor on it.

  

- **Invariant: “next” is either nullptr or a valid pointer to a heap-allocated object**
    
    - An invariant is a statement that must hold true upon which our class relies
        - Options:
            - trust user (not reliable, kinda of don’t trust them)
            - use c++ to enforce the invariant, example:
    
      
    

```C++
class Stack {
	// Invariant in this situation: last item pushed onto the stack is the first
	// item to be popped off of the stack 
```

In a similar way, the Node code above has something that violates its invariant. For stack, you can imagine someone going in and change the order of the stack. Of course it’s possible, but it should not be “possible”. We should only allow the client to `push` and `pop` from the stack, we should never give them control, hence why we need `private` and `public` parameters for `classes`.

  

**Encapsulation**

- Our objects are black boxes, capsules, with hidden implementation details
- The user manipulates the data via provided methods, sort of like an `API`

```C++
struct Vec {
	Vec (int x, int y);

	private: // what follows is private, cannot be accessed by the client, 
					 // outside of Vec, so if we put "data" in private, 
					 // we cant directly access it

		int x,y;
	public: // what follows is public, can be accessed by all, 
					// whoever creates object can 
					// access this data
		Vec operator+(const Vec&other);
		...
}
```

- For `class`, by default, everything is private
- For `struct`, by default, everything is public

**OK, WE CAN FINALLY USE CLASS KEYWORD**

```C++
class Vec {
	int x,y; // these are private 
	...
	... // all of the private data
	public:
		Vec(int x, int y);
		Vec operator+(const Vec &other) ;
}
```

So, now let us jump back to the first chunk of code for this lecture, pasted below for convenience:

---

**Goal: Program creates lists with our Node class**

```C++
\#include <iostream>

Node plusOne( Node n ) {
	for ( Node *p{&n}; p ; p = p->next ) {
	++p->data;
	}
	return n; 
}

int main() {
	Node n {1, new Node {2, nullptr}};
	Node n2{plusOne(n)};
}
```

**Solution: Creating a wrapper class list that has exclusive access to underlying Node objects**

- The above “wrapper class” is an instance of `inheritance`

  

**Solution:**

```C++
// list.h 
class List { // Would need a constructor for List 
	struct Node; // private nested class, who is by defualt public for this class
	Node *theList = nullptr; // pointer for the beginning of the list

	public:
		void addToFront(int n);
		~List();
		... // other things in class
}

// list.cc
struct List::Node { // since this is private in List, user cant touch, even if 
										// it is a struct (def. public)
	int data;
	Node *next;
	
	Node(int data, Node *next): data{data}, next{next} {}
	~Node () {delete next;}

}

List::~List() {delete theList;} // user must make sure rest of list is deleted

Void List::addToFront(int n) {
	theList = new Node (n, theList);
```

Now, lets look at a function for Node

```C++
//ith(int i) returns the data of Node at position i in the list 

int List::ith(int i) { // so ith is a member function of List 
	Node *cur = theList; 
	for (int j = 0; j < i; j++) {
		cur = cur->next;
	}
	
	return cur->data;
}
```

**Problem:**

- The user depends on `List` to iterate through the `Nodes`
- We could expose `Node` to allow client to directly iterate, but exposing `Node` would break the encapsulation

**Solution: Iterator Pattern**

- Creates a `class` that manages the access of `Node`, and works basically as an abstraction of a pointer
- Iterator class is an abstraction of a pointer
- Let the user talk through the list without exposing the `Node` class

```C++
class List {
	struct Node:
	Node *theList;
	
	public:
		class Iterator { 
			Node *p;
			
			public:
				explicit Iterator(Node *p): p{p} {} // constructor 
				int &operator*() {return p->data;} // access the item (dereferencing)
				Iterator &operator++ () { // advanced pointer method 
					p = p->next;
					return *this;
				}

				bool operator==(const Iterator &other) const { 
				// adding const at the end specifices the operator won't be changing 
				// the data fields of the current object, or "this"

				// const in the parameter says the pass in paramater will not be changed
				// outside one says the data field of the object wont be changed 
					return p==other.p;
				}
				
				bool operator!=(const Iterator &other) const {
					return !(this==other);
				}
```