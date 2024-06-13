- Advice: Prefer `pass-by-const-ref` for anything larger than an int, unless you need to make a copy anyways, then perhaps use `pass-by-value` (but not always)
- Also:
    - `inf f (int &n) {...}`
    - `int g (const int &n) {...}`
- What would happen if I did `f(5)`?
- There would be a compiler error, as we passed in an l-value but we're expecting a a reference to the literal. If n changes, what does it mean to change 5?
- On the other hand, `g(5)` is fine as we've promised the compiler that g will not change the parameter

How can `g(5)` work if `n` must be bound to something with a memory address? The compiler creates a *** temporary location *** in memory to hold the `5`, and lets `n` refer to that.

---

### Dynamic Memory Allocation

In C++, we use:

```C++
struct Node {
  int data;
  Node *next;
}

Node *np = new Node;
...
delete np;
```

**Reminder**:

- All local variables reside on the stack.
- Variables are deallocated when they go out of scope (stack is popped)
- Allocated memory exists on the heap
- Remains allocated until `delete` is called
- If you don't delete all allocated memory - memory leak
    - Program will fail eventually
    - We regard any memory leak in this class as an `error`

**Best way** to check for memory leaks is using the tool `valgrind` on your program

- e.g. If your program is called `prog` (in your `cwd`):
    - `valgrind ./prog` or replace `prog` with the path if `prog` is not in `cwd`
    - Don't return references or pointers to local stack frame variables (common sense)

---

### Initializing Array Forms

```C++
Node *myNodes = new Node[10]; // Array of 10 nodes on heap
...
// to delete, we do:
delete[] myNodes; // note the square brackets on delete
```

- It is important to match the correct form of delete with the the corresponding `new`
- Every call to `new` should have a corresponding call to `delete`
- Every call to `new[]`, should have a corresponding call to `delete[]`

**Returning by value/reference/pointer**

```C++
Node getMeANode () { // possibly expensive, it is copied onto the caller's stack frame
  Node n;
  return n;
}
```

If this is expensive, return a `pointer` or a `reference`instead.

```C++
Node *getMeANode() {
  Node n; // BAD - returns a pointer to stack-allocated data that ceases to be allocated for this purpose when this fn returns
  return &n;
}

Node *getMeANode() {
  Node. *np = new Node; // this is okay, returns a pointer to heap-allocated data, but caller is now responsible for deleting it
  return np;
}
```

Which should you pick in general? Depends on your needs, but return by value (unless you actually wanted to keep allocated data) because it won't be as slow necessarily.

---

### Meanings to C++ operators for own types

- We can give meanings to the `C++` operators for our own types

**Operator Overloading**

```C++
struct vec {
  int x,y;
};

vec operator+(const vec &v1, const vec &v2) {
  vec v{v1.x+v2.x, v1.y+v2.y};
  return v;
}

vec operator*(const int k, const vec &v) {
  vec v{k*v.x, k*v.y};
  return v;
}
```

This is an example of overloading the `+` and `*`operators for my own `structures`.

```C++
vec a{1,2};
vec b{3,5};
vec c = a+b;
vec d = 5*a;
vec e = d * 2; // not defined as I defined above int * vec, not vec * int

// to make vec e work, we need to overload + operator again
vec operator*(const vec &v, const int k) {
  return k*v;
}
// notice how I can use another operator I defined to defined this new operator
```

**Two of the most useful operators to overload are the IO operators**

Overloading `input` and `output`operators are very handy

```C++
struct Grade {
  int theGrade;
};

// output operator, returns ostream reference
ostream &operator<<(ostream &out, const Grade &g) {
  out << g.theGrade << "%";
  return out;
}

istream &operaotr>>(istream &in, Grade &g) {
  in >> g.theGrade;
  if (g.theGrade < 0) {
    g.theGrade = 0;
  } else if (g.theGrade > 100) {
    g.theGrade = 100;
  }

  return in;
}
```

### Next Time:

- ** The Preprocessor ***

The preprocessor transforms the program before the compiler sees it!