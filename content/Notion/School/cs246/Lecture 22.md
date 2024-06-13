# Lecture 22

```Plain
// window.h
class Xwindow {
  Display *d;
  window w;
  int s;
  GC gc;
  unsigned long colours[10];
  ...// some other stuff
};
```

If we change a private member, then all client code must recompile. Would be better to hide these details away.

Solution - use the pointer to implementation, known as the `(pImpl)` idiom.

So, let's convert out above snippet into `pImpl` idiom.

```Plain
// windowImpl.h
\#include <X11/xlib.h>
struct XwindowImpl {
  Display *d;
  window w;
  int s;
  GC gc;
  unsinged long colours[10];
  ...// some other stuff
};

// don't even need a .cc file for this

// window.h
class Xwindow {
  Xwindow Impl *p;
  public:
  // no changes from before
};

// size of Xwindow object never changes, always one pointer large

// window.cc
\#include "window.h"
\#include "windowImpl.h"

Xwindow::Xwindow {...}: p{new XwindowImpl{...}} {}

// all other methods just change field accesses from d,w,...etc. to p->d, p->w, etc.
```

If you confine all private fields within an implementation `struct`, then your actual class never needs to change size and changes to the private data means only your class, not client code, needs to recompile.

---

### Measures of Design Quality

**Coupling and Cohesion:**

**Coupling:** How much distinct program modules depends on each other. Don't really want highly coupled software:

1. (low) Modules communicate with function calls with basic parameters and results
2. (higher) Modules pass arrays or `struct`s back and forth
3. (higher) Modules affect each other's control flow
4. (Quite high) Modules share global data
5. (Very high) Modules have access to each other's implementation (friend is indicative of high coupling, but sometimes, they are necessary. Friends, like iterator, for example, are fine. But, friends across modules are iffy)

High coupling:

- Changes to one module require greater changes to compiled modules
- Harder to raise individual modules
- *A good design will often include things like re-usability, easy changed, etc. **

**Cohesion:** How closely elements of a module, are related to one another.

1. (low) arbitrary grouping of unrelated elements (e.g. the standard library `<utility>`, nothing is related to each other. All of functions in it are just small things that are helpful, low cohesion)
2. (better) Elements share a common theme, but are otherwise unrelated (e.g. the `<algorithm>`)
3. (better) Elements manipulate state over the lifetime of an object (e.g. open/read/close files)
4. (high) Elements cooperate to perform exactly one task (a good example, is `<vector>` library.)

**low Cohesion:**

- Poorly organized code
- Hard to maintain/understand

**Goal:** **low coupling** and **high cohesion**

Let's take a look at an example: I want a chessboard class

```Plain
class ChessBoard {
  ...
  ... cout << "your move" << endl;
  ...
};
```

- This is bad design, inhibits code reuse. What if you wanted a chess board, but don't want to communicate to standard out
- You could pass in the stream you want to communicate through but, what if you don't want to communicate through text at all?
- Chess board should not be handling communication at all!
- * Single Responsibility Principle**: A class should only have one reason to change.

However, for the chess board above, there are 2 reasons: game logic, and communication. A chess board should be a `chessBoard`, a separate display class can interact with the public interface to display the board (`textDisplay` or `graphicsDisplay`).

---

### Exception Safety

Consider:

```Plain
int **f(int s, int (*g)(int)) {
  int **p = new int*[s];
  for (int i = 0; i < s; i++) {
    p[i] = new int{i};
    *p[i] = g(*p[i]);
  }
  return p;
}
```

What happens if `g` or `new` throws? If the first `new`, the one creating the initial array, fails, no problem. However, if any of the inner `new`s or `g` fail, we have a memory leak, as we have allocated a "partial" array.

Now, let's change the code above:

```Plain
int *8f(int s, int (*g)(int)) {
  int **p = new int *[s];
  for (int i = 0; i < s; i++) {
    try {
          p[i] = new int{i};
    *p[i] = g(*p[i]);
    } catch (...) {
      for (int j = 0; j < i; j++) {
        delete p[j];
      }
      delete p;
      throw;
    }
  }
}
```

However, this is wrong. In fact, catching an error like this, using try catch method inside a function where mutliple places can have errors, is not good. In the example above, if `g` fails, not `new`, we must free up to and including `i`, not just up until `i`.

Some languages have a "finally" clause, in which you write code guaranteed to run no matter how the function exists, not true in c++.

All c++ guarantees is that the destructors for stack allocated objects will run, so...

**C++ Idiom: RAII**: Resource Acquisition is Initialization

The RAII states you should only ever acquire resources as a result of initializing a stack allocated object whose job it is to manage that resource.

**BONUS MARKS ON A5 FOR NOT USING NEW OR DELETE, USE RAII**  
So, lets re-write our example, to follow RAII:  

```Plain
vector<unique_ptr<int>> F(int s, int (*g)(int)) {
  vector<unique_ptr<int>> v;
  for (int i = 0; i < s; i++) {
    v.emplace_back(Make_unique<int> (i));
    *v[i] = g(*v[i]);
  }
  reutrn v;
}
```

Class `std::unique_ptr<T>` is a class that holds a `T*` which you supply in the constructor. For example, `unique_ptr{new int {10}` **OR**, `Make_unique<T>` returns a `unique_ptr<T>`, and takes as args the constructor parameters or initializing data for T.

`unique_ptr` and `Make_unique` are in `<memory>`. The destructor `unique_ptr` frees the pointer, inbetween you can dereference the `unique_ptr` just like a `ptr`.

`unique_ptr`s are unique...

```Plain
unique_ptr<c> p{new c{...}};
unique_ptr<c> q = p; // error!
```

`unique_ptr` have **NO** copy constructor **OR** copy assignment operator. Can only be moved from.

However, you can still use raw pointers for non-owning pointers, but must be careful.

```Plain
void foo(int *p) {
  *p = *p + 5;
}

int main() {
  unique_ptr<int> p{new int{10}};
  foo(p.get());
  // unique_ptr<T>.get() returns the underlying raw pointer
}
```

Beware!

```Plain
int *p = nullptr;
if (...) {
  unique_ptr<int> q{new int{10}};
  p = q.get();
}

*p = *p + 5;
```

Be careful!. Once we leave `if` statement, `q` goes out of scope, and owns the pointer, so it deletes it. But now, p is a pointer to invalid memory. The lifetime of raw pointer must be shorter than the lifetime of the unique_ptr.