# Lecture 17

Last time, we left off on `virtual`.

A C++ virtual function is a member function in the base class that you redefine in a derived class. It is declared using the virtual keyword. It is used to tell the compiler to perform dynamic linkage or late binding on the function.

We don't need `override`keyword, it just explicitly tells compiler to check if there is matching virtual method in base class.

```Plain
Book *arr[20]; // array of book pointers
arr[0] = new Book {...};
arr[1] = new Text {...};
// the above is legal, but won't work completetly at runtime

// will call book versions of all objects put in, not text/comic

for (int i = 0; i < i++) {
  cout << myBook[i]->isheavy() << endl;
}
```

This code executes `Book:isheavy()`for Books, `Text::isheavy()`for Texts and `Comic::isheavy()` for comics.

The code acomodates multiple types under one abstraction(the Book abstraction) - polymorphism

Note: This is why our input operator rakes an `istream &` but can operate on `ifstream`, `istringstreams`, etc.

**Destructor Revisitd**

```Plain
class X {
  int *x;
  public:
    x(int n) : x{new int{n}} {}
    ~x() {delete x;}
};

class Y: public X {
  int *y;
  public:
    y(int m, int n): x{n}, y{new int{m}} {}
  ~y() {delete y;}
};
```

so, destructing y runs into some problems:

```Plain
X *myx = new Y{0,20};
...
delete myx;
```

Here, `Y`'s destructor never runs. So memory leak. To prevent this, we need to make `X`'s destructor virtual.

```Plain
class X {
  int *x;
  public:
    x(int n) : x{new int{n}} {}
    virtual ~x() {delete x;}
};
```

Now, this code no longer leaks.

**Quick summary**

- When dtor is not virtual, this only calls `X::~X()`, if it is virtual, then `Y::~Y()` is called, as part destroying that `Y`, its superclass component is destroyed and `X`'s dtor runs as well
- So, if you have **inheritance**, your destructor should **ALWAYS BE VIRTUAL**, even if the base class doesn't have resources to manage, the derived classes later might.
- If no inheritance, no need for `virtual`

Now, lets revisit student class:

```Plain
class Student {
  ...
  protected:
    int numCourses;
  public:
    virtual int fees() const;
  ...
};
```

There are two kinds of students, regular and co-op.

```Plain
class Regular: public Student {
  ...
  public:
    int fees() const override; // need const if Student has it
  // compute fees fr reg student
};

//////////////////////////

class Coop: public Student {
  ...
  public:
  int fees() const override;
  // compute fees for coop student
};
```

But, what should we put for `Student::fees()`?  
... Not sure:  

- Every student should be regular or coop, there should be not student objects.

So, we do this in the student class:

```Plain
class Student {
  ...
  protected:
    int numCourses;
  public:
    virtual int fees() const = 0;
  ...
};
```

- Here, `Student::fees()`is a pure virtual method, which is a method that does not require an implementation. A class with a pure virtual method is called a **Abstract** method and cannot be instantiated.
- As long as there is **ONE**, pure virtual method, your class is **Abstract**
- e.g. `Student S; // error`

The purpose of abstract classes is to define common interfaces and organize subclasses. Subclasses of an abstract class are also abstract unless they implement pure virtual methods.

Note:

- Abstract classes are called concrete
- In **UML**, visualize abstract classes and pure virtual methods by italicizing their name

### Templates

Massive, but we'll only going to go through only the basics

```Plain
class List {
  Struct Node;
  Node *theList;
  ...
};

// what if we want to store something other than ints
// in our list?

// A template class is a class parameterized by a type

// We start by
template <typename T> // some type T
  class Stack {
      int size;
      int cap;
      T *contents; // arbitrary type array
    public:
    Stack () {...}
    void push(T x) {...}
    T top {...}
    void Pop() {...}
  };
```

Now, lets revisit List as a template class

```Plain
template <typename T>
  class List {
     Struct Node {
       T data;
       Node &next;
     };

    public:
      class Iterator {
        Node *p;
        ...
       public:
        ...
        T &operator*() {return p->data;}
      };

    void addToFront(T x) {
      theList = new Node{x, theList};
    }
  };
```

Note, that the above is exactly the same as the one learned before, but, insetad of having `int`, we just changed it with a type `T`, which can be anything.

```Plain
List<int> l1;
List<char> l2;
List<List<int>> l3;

// with the template, we can now use implementation to
// accept ANY data type for List
l1.addToFront(3);
l3.addToFront(l1);

for (List<List<int>>::iterator it = l3.begin(); it!=l3.end(); i++_) {
  for (auto n; *it) {
    cout << n << " ";
  }
  cout << endl;
}
```

So, **HOW**do templates work? Roughly speaking, the compiler sees each initialization of the class (e.g. `List<int>, List<List<int>>, etc`and generates specialized classes for each type declared.

**TEMPLATE CODE MUST BE IN HEADER CLASS, CAN'T BE IN CC FILE**

Compiler needs to know before linking, of the types `T` can take. The restrictions on `T` are what I, as the coder, make them.

---

### This brings us to the STL (Standard Template Library)

- The STL is a large collection of useful templates
    - Example: dynamic-length arrays
    - Thank god, we can use vectors now

```Plain
\#include <vector>
using namespace std;

vector<int> v{4,5}; //vector containing (4,5)
vector<int> w(4,5); // different from above, this one
                    // contains 4 fives (5,5,5,5)

v.emplace_back(6); // (4,5,6)
v.emplace_back(7); // (4,5,6,7)

// I don't care out managing memory, vec does this for me
```

Now, consider the following:

```Plain
{
  vector<int> c{3,2,1};
}

// v goes out of scope, array is freed, noting leaked
// BUT!

{
  vector<int *> x{new int{1}; new int{2}};
  for (auto &p; w) delete p;
}

// NOTE: iterator is supported by vec
// Iterator is a huge component of C++, very important
// in achieving abstraction
```

Looping over vector:

```Plain
for (int i = 0; i < w.size(); ++i);
  cout << w[i]; << endl;
}
// vectors also support the iterator abstraction!

// so, if I have vector of int v
for (vector<int>::iterator it; it != v.end(); ++it) {
  cout << &it << endl;
}

// or indeed:
for (auto n: v) {cout << n << endl;}
```

We can also iterate in reverse:

```Plain
for (vector<int>::reverse_iterator it = v.rbegin(); it!= rend(); ++it) {
  ....
  }

// this iterates in reverse, cannot range base this

v.pop_back() // removes the last the element of v
```

Vectors are guaranteed to be implemented as arrays internally. Now that we have vectors, basically never use dynamic arrays again.

Many other vector operators are based off iterators, e.g. the erase method, which takes an iterator to the element you want to remove.

Be careful when using iterator and vectors. If we have an iterator pointing to data, and then we add something to the vec, the vec reallocates itself. But then, since we delete old array after copying it into a new one, iterator now points to invalid memory.

What are some ways we an initialize vectors?

```Plain
// Create a vector of size n with
// all values as 10.
vector<int> vect(n, 10);

// vector with the numbers in it, just like array
vector<int> vect{ 10, 20, 30 };

// empty
vector<int> vect;

// vector of size 10, with default values for the type
// e.g. 0 for int, \\0 for char, something for string,
// nullptr for pointers, etc.
vector<int *> mois (10);


//!!!!!!!!!!!!!!!!!!!!!!!!!
// remember the difference between () and {}
vector<int> allan (10,10); //vector of length 10, all 10s
vector<int> allan{10,10}; // vector of length 2, 2 10s
```