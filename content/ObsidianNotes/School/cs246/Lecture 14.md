Note: write `explicit` at the beginning of constructor if it only takes in one value

---

This little blob contains some after class notes I wanted to take regarding Iterators.

- `Iterator` is actually a standard library class, and it helps us move through containers
- It is similar to how we can move through an array using pointers and the `[]` notation.
- Imagine that you have the code below:

_Note: No need to fully understand code below, from internet, just want to demonstrate_

```C++
int ComputeSum(const std::vector<int>& dynamic_array) { 
  int sum = 0; 
  for (int i = 0; i < dynamic_array.size(); i++) { 
     sum += dynamic_array[i]; 
  } 
  return sum; 
} 
 
int ComputeSum(const MyLinkedList<int>& my_linked_list) { 
  int sum = 0; 
  MyLinkedList<int>::Node* node = my_linked_list.first(); 
  while (node != nullptr) { 
    sum += node->value(); 
    node = node->next(); 
  } 
  return sum; 
}
```

Versus:

```C++
template<typename T> int ComputeSum(T begin, T end) { 
  int sum = 0; 
  while (begin != end) { 
    sum += *begin; 
    begin++; 
  } 
  return sum; 
}
```

  

- Here, we need two separate functions to compute the sum of the data in a linked-list and a dynamic array.
- The way each data structure is structured is different, hence why we have two different functions
- However, the logic behind how we compute this sum is nearly identical
- We iterate through the structure at each “node”, and add each data into a sum
- Iterators provide us a level of abstraction, to be able to design a way of computing sum in once function, that is capable of taking in either data type
- Here’s another way to look at it
    - Integers have `++` to increment 0 to 1
    - Pointers have `++` to increment from one bit in memory to a next one
    - These capabilities allow us to iterate through integers in a for loop, or through an array using incrementing pointers
    - Wrapping our “list” class with an iterator class gives us the power to define a new `++` for the class we are working with

  

_END OF OWN NOTE, RESUMING LECTURE NOTES_

---

  

```C++
\#ifndef _LIST_H_
\#define _LIST_H_

class List {
 private
	 class Node;
	  Node *theList = nullptr;

 public:
  class Iterator {
    Node *p;

    explicit Iterator(Node *p); 

   public:
    int &operator*() const {
			return p->data;
		}

    Iterator &operator++() {
			p = n->next;
			return *this;
		}

    bool operator==(const Iterator &other) const {
			return (p == other.p);
		}

    bool operator!=(const Iterator &other) const {
			return !(p == other.p); // could also do p != other.p
	}

    friend class List; // List has access to all members of Iterator
  };

	// list.begin() => returns an Iterator pointing to the beginning of the list 
  Iterator begin() {
		return Iterator(theList); // remeber theList is pointer to first Node 
	}

	// list.end() => returns an Iterator pointing to the end of the list 
  Iterator end() {
		return Iterator(nullptr);

  void addToFront(int n);
  int ith(int i);
  ~List();
};

\#endif

// Go over this code again!!!!!!!!!!!!!!!!!!!!!!!!!!!!

int main() {
	List lst;
	lst.addToFront(1); // lst: 1
	lst.addToFront(2); // lst: 2,1
	lst.addToFront(3); // lst: 3,2,1

	for (List::Iterator it = lst.begin(); it!=lst.end(); it++) {
	// note that the things in the for loop, are the operations we have defined 
  // in our class
	std::cout << *it << std::endl;
}
// 3
// 2
// 1
```

`c++` has automatic type deduction. By using the keyword `auto`, we can define a variable without knowing its type.

```C++
auto x = y;
```

So, the compiler checks the type of `y`, and provides that same type to `x`. If y was an object, then of course, compiler would call on the copy constructor to build `x`.

  

So, we could have used the following for loop in the above code:

```C++
for (auto it = lst.begin(); it!=lst.end(); it++) {
	std::cout << *it << std::endl;
}
```

Another way to make this for loop shorter and cleaner, is using a `range-based` for loop. When do we use range-based for loops?

- Methods begin and end (imagine an array, is best example, or linked list, etc.)
- Iterator supports:
    - operator ++
    - operator !=
    - operator *

```C++
// for a ranged based for loop
int allan[3] = {1,2,3};
// to loop through with range-based, is easy
for (auto num : allan) { // auto makes num an int
	cout << num << endl;
}
// This sort of loop makes a copy of each int and prints i
// This can be very expensive if we use objects 

for (auto &n : lst) {
	n++; // mutates the actual object 
}
// AGAIN REVIEW THIS, n should be a Node?? or int 
```

---

There are different ways we can provide ways to access a classes’ data:

```C++
class Vec{
	int x,y;
	
public:
	int getx() const {return x;} // accessor 
	void sety(int z) {y = z;} // setter
	friend ostream&operator<<(ostream &out, const Vec &v);
```

Now, consider another example:

```C++
ostream &operator<<(ostream &out, const Vec &v) {
	return out << v.x << v.y;
}
// so this function is a friend to that class, so it has access to the class data
```

When we have this function as a member of the above `Vec` class

  

**So, everything up until this point will be on the midterm. Assignment 3 is really good practice for midterm, do it ASAP.**

---

```C++
Vec v{1,2};
Vec v1 = v*2;
// we learned how to overload operators like this
Vec v2 = 2*v; // needed another for this 
```

Again, consider the followig:

```C++
class Vec {
	int x,y;
	// some other stuff ... 
	Vec operator*(const int k) {
		return {x*k, y*k};
	}

// Now this operator overloading works fine with v *3, or v * (some int)
// but doing (some int) * vec, won't work as it doesn't match the overloaded 
// operator 
```

**Standalone function:** Defined outside of your class

```C++
Vec operator*(const int k, const Vec &v) {
	return v*k;
}
// now, we can do 5*vec 
```

Now, let us jump back to the `Vec` class we defined earlier:

```C++
class Vec{
	int x,y;
	
public:
	int getx() const {return x;} // accessor 
	void sety(int z) {y = z;} // setter
	// friend ostream&operator<<(ostream &out, const Vec &v);
  // ostream &operator 

	// say instead of the commented out code above, we had
	ostream &operator<<(ostream &out) {
		out << x << " " << y;
	}
	
	// this is a problem, since if we wanted to print our vector with the above 
	// member function, we would need to write v << out, very counterintuituve 

	// this is why operators should be standalone functions 
```

**Again, this needs review, a tad confusing**

---

**Operators should be standalone functions, e.g:**

- operator <<
- operator >>

  

Now, there are certain operators that can, and should, be members of a class:

- operator =
- operator →
- operator []
- operator ( )
- operator T( ), with T being a type, type casting

The above are just some examples, know of them, **don’t** need to know in depth how they work

  

---

**Declaring arrays of objects**

Say we wanted to initialize an array of `Vec` objects:

```C++
Vec *vp = new Vec[10];
Vec moreVectors[15];

// now, if our class has only the costructor
// Vec(int x, int y) : x{x}, y{y} {}
// How can we solve this? 
```

So, how can we solve this? There are a couple of options:

1. Define a default constructor (so a constructor that has nothing passed to it)
2. Initialize every element during the declaration (for stack allocated arrays)
3. Create arrays of pointers (for heap arrays), where each pointer points to ab object which we can define later in the code

  

**Example of sol.2 :**

```C++
moreVectors[3] = {Vec{0,0}, Vec{1,3}, Vec{2,4}};
```

**Example of sol.3 :**

```C++
Vec **vp = new Vec *[5];
...

// later in the code
vp[0] = new Vec{0,0};
vp[1] = new Vec{1,3};

...
// later in the code 
for (int i = 0; i < 5; i++) {
	delete vp[i];
}
delte [] vp;
```

Note that we need to delete like this if we are using the default destructor, experiment with this. With a user-defined destructor, could be done inside the destructor.

---

Now, let us revisit the `Student` class:

```C++
//student.h
struct Student{
	int assns, mt, final;
	float grade() const; 
  // only want to return student grade, wont be changing object data
};
	
//student.cc
float Student::grade() const { 
	return assns*0.4 + mt*0.2 + final*0.4;
}
```

Say we want to see how many times we call `grade()`. We can do the following:

```C++
// add this into the Student struct 
int numMehtodCalls = 0;

// so, we make a small change to grade function 
float Student::grade() const { 
	numMehtodCalls++; 
	return assns*0.4 + mt*0.2 + final*0.4;
}
// UH OH, we have now changed the data of the object, but we wrote const 

// But its not like this data change affects the purpose of the class, its 
// something we added as developers to regulate code, so we can add the key word 
// mutable, to tell compiler its fine if this changes in the const function

// so, now we get
struct Student{
	mutable int numMehtodCalls = 0;
	int assns, mt, final;
	float grade() const; 
  // only want to return student grade, wont be changing object data
}; 
```