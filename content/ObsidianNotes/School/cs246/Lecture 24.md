# Lecture 24 - FINAL LECTURE

### Casting

In `C`:

```Plain
Node nl
int *ip = (int *)&n;
// a cast, forces c to treat a node * as an int *
```

Casts should be avoided. However, if you must need to cast, you should use `C++` style casts instead of `C`.

`C+` casts come in 4 varities:

### 1: static_cast:

- is for "sensitive casts" with well defined behavior. (e.g. a double to an int)

```Plain
double d{...};
void f(int x);
void f(double d);

// to cast to the int version, we can:
f(static_cast<int> (d));
```

- Superclass pointer to a subclass pointer, BUT, we must know it actually points at that object:

```Plain
Book *b = new Text{...};
text *t = static_cast<Text *> (b);

// you are taking responsibility that b actually points at a text, essentially you are telling compiler, trust me

// if it doesn't point to text, undefined behavior
```

### 2: reinterpret_cast:

- is for unsafe, implementation dependent, "wierd conversions", almost all uses of `reinterpret_cast` result in `undefined_behavior`

```Plain
Student s;
Turtle *t = reinterpret_cast<Turtle *> (&s);

// forces student to be treated like a turtle
// again, undefine behavior, if doing this, could

t->beStruckBy(stick{}); // strike student with stick... lul
```

### 3: const_cast:

- is for converting between `const` and `non-const`. It is the only `C++` style cast that can cast "cast away constness"

```Plain
void g(int *p); // you KNOW g doens't actually modify *p
void f(const int *p) {
  ...
  g(const_cast<int *> (p));
  ...
}
```

On the other hand, if `g` does change `*p` above, this is very bad!!

### 4: dynamic_cast:

- is it safe to convert a `Book *` to a `Text *`. From superclass to subclass.

```Plain
Book *pb = ...;
Text *t = dynamic_cast<Text *p> (pb);
// is this safe? Only if pb actually points at a text
```

- `dynamic_cast<T*< (p)` returns `p` (as a T pointer) is `p` actually points at a `T`. Otherwise, returns `nullptr`.
- Works by looking at the virtual pointer of that object.
- This only works on hierarchy with at least one virtual function

---

**Generally, using casting is indicative of poor design**

```Plain
void whatIsIt(Book *pb) {
  if (dynamic_cast<Text *> (ob)) {
    cout << "Text" << endl;
  } else if (dynamic_cast<Comic *c> (pb)) {
    cout << "Comic" << endl;
  } else {
    cout << "Book" << endl;
  }
}
```

This is poor design, as it is highly coupled to the book hierarchy. Defeats the purpose of polymorphism.

But, all of these operations are on raw pointers, can we do equivalent operations on smart pointers? Yes

- `static_pointer_cast`
- `const_pointer_cast`
- `dynamic_pointer_cast`

Dynamic casting also works on references:

```Plain
Text t{...};
Book &b = t;
Text &t2 = dynamic_cast<Text &> (b);
```

If `t` actually refers to a `Text`, then the `dynamic_cast` returns a `Text &` to it, if now, no such thing as "null reference", so it raises the exception `bad_cast`.

With `dynamic_cast`, we can (if we want), implement a polymorphic assignment operator.

```Plain
Text &operator=(const Book &b) {// virtual in base class
  if (this = &t) return *this;
  Book::operator=(t);
  topic = t.topic;
  return *this;
}
```

But, this hasn't really solved the problem, just passed the book onto the client, who must now handle the exceptions raised by mixed assignment through the class pointers/references. It's still true that polymorphic solution is still as it was before. Prefered solution should be as it was before, all base classes should be abstract, and make the assignment operator protect it in the base class.

---

Some personal notes:

```Plain
class Asset {
  virtual get_name() = 0;
}

class Money: public Asset {
  static string name = cash;
  public:
  string get_name() {return name};


}

class Ownable: public Asset {
  string name;

  public:
  string get_name() {return name};

}

// work polymorphically with assets in player, and with
// buildings in board

// REMEMBER!
```

---

### Multiple Inheritance

```Plain
struct A {
  int a;
};

struct B: public A {
  int b;
};

struct C: public A {
  int c;
};

struct D: public B, public C {
  int d;
};

D d;
d.a = 15; // error, d has two a's, one from B, one from C

d.b::a = 15;
d.c::a = 15;
// but, what do these two a fields represent
// not solved with ::, we want only one a

// this problem arrises when inherting from two base classes
// that have common ancestor
```

This problem is called the **THE DEADLY DIAMOND**, since thats what the UML looks like.

What we really want is a singular A field, that represents our A component. We can achieve this through **virtual inheritance**:

```Plain
struct A {
  int a;
};

struct B: public virtual A {
  int b;
};

struct C: public virtual A {
  int c;
};

struct D: public C, public B { // only need virtual inheritance
  // is somehting will inheirt from D and may have same issue
  int d;
};

D d;
d.a = 15; /// works now!

vfunciton table ?
```

### Template Functions

```Plain
template<typename T>
T min(T x, T y) {
  return x<y? x:y;
}

// Now, I can call min on vairous types
int f() {
  int x = 1, y = 2;
  int z = min(x,y);
  // T = int
}
// notice that here, we didnt say min<int> (x,y), I COULD say
// that. But, if the compiler can deduce the template types (typically if your template types are your parameters, it can deduce based on any type), so if it can be, don't need to excplicitly specify.
```

Template functions operating on iterators are very effective c++. e.g:

```Plain
template (Typename Iter, typename Func>
void for_each(Iter start, Iter end, Func f) {
  while (Start != end) {
  f(*(iter__));
  }
}
```

e.g.

```Plain
void print(int n) {cout << n << endl;}
int p[] = {1,2,3,4,5};
for_each(p, p+s, print);
```

`func` must be callable (specifically on the type produced by *iter)

But, what is callable other than a function? Anything that overloads `operator()` (The function call operator)

```C++
class Plus {
  // we can write functions that rdeturn functions now
  int toAdd;
  public:
  Plus(int toAdd):  toAdd{toAdd} {}
  void operator() (int &n) {
    n = n + toAdd;
  }
};

Plus add5{5};
for_each(p, p+5, add5); // adds 5 to prev array
Plus sub3{-3};
for_each(p, p+5, sub3); // subtracts 3 from each element in p
```

Last notes:  
  
`for_each` and functions like it already exist! They're in the `<algorithm>` header and most of them operate on iterators. Teacher is **strongly suggesting** I familiarize myself with this library and what it does. Not needed for course/exam, but can be very useful, very similar to things in `cs135`. Things such as `foldr` and `foldl`.

```Plain
for_each(p, p+s, [] (int &n) {n = n + s;});
```

`[]` is the lambda specifier.

### COURSE IS DONE, CLOSED