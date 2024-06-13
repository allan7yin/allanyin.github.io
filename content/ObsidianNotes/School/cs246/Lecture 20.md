# Lecture 20

Recall:

```Plain
void F() {
  throw out_of_range{"f"}; // "f" will be returned by out_of_range
}

void g() { f(); }
void n() { g(); }
int main() {
  try {
    n();
  }
  catch (int_of_range) {cout << e.what() << endl;}
```

- What Happens? Main calls h, h calls g, g calls f then raises out_of_range.
- Control goes both through the call chain (unwinds the stack) until a handler is found. In this case, goes all the way back to main and main hanldes the exception.
- If no matching handler is found, the program terminates.
- Out_of_range is a class. The statement throw out_of_range {"f"} constructs the object and throws it as an exception.
- A hanlder can do part of the recovery job and throw another execption.

e.g.

```Plain
try {...}
catch (someErrorType s) {
  ...
  throw SomeOtherError{...};
}

// or we can rethrow same exception
try {...}
catch (someErrorType s) {
  ...
  throw;
}
```

Why `throw;` vs `throw s;`?

[SomeErrorType] <- [SpecialErrorType]

The exception `s` might actually belong to a subclass of `someErrorType`, rather then `someErrorType` itself. `throw s;` would throw the local copy `s`, which is a sliced version of the original exception. On the other hand, `throw;` rethrows the actual exception that was ?, and its type maintained.

A hanlder can act as a catch-all

```Plain
try {...}
catch (...) {// literlly mean ... here
   ...// figureatievly mean ... here
  }
```

Technically, you can throw anything you like, not just objects. But, generally, exceptions should only be used for exceptional cases (errors that arise). You typically want a meaningful error class so define your own or use an appropriate one.

```Plain
class BadInput {};
try {
  int n;
  if (! (cin >> n)) throw BadInput{};
} catch (BadInput &b) {
  cerr << "input not well formed " << endl;
}
```

- Not that here, exception was caught by reference. This prevents the exception from being sliced (if it actually was sliced)
- Catching exceptions by `ref`is usually the right thing to do, the maxim c++ is "throw by value, catch by ref"

When `new`fails, it throws `std::bad_alloc`. Warning! NEVER let a destructor throw, by default, the program will terminate immediately because the compiler implicitly marks `destructors`as functions that should never throw.

It is possible to create a throwing destructor by overruling this implicit devision. But you still **should NOT** do this, if a destructor throws as a result of it executing due to stack unwinding cause by another exception, you now have **two** active exceptions. Program will abort immediately, no other objects further down the call chain get cleaned up.

Now, let's move onto another design pattern.

---

### Factory Method Pattern

- *Problem: ** write a video game with two types of enemies, `turtles` and `bullets`, the system randomly sends `turtles` and `bullets`, but Bullets become more frequent in later levels.

So, we have abstract base class `Enemy` with `turtle` and `bullet` inheriting from it.

Then, we have abstract base class `Level`, with `NormalLevel` and `Castle` inheriting from it.

Since we never know exactly what enemy comes next, we don't want to call constructors directly, moroever, we don't want to hardcode the decision policy, as we want it to be customizable.

This pattern is basically just using a **virtual function**.

So instead, we put a Factory Method in `Level` that creates enemies.

```Plain
class Level {
  public:
    virtual Enemy* createEnemy() = 0; // pure virtual
    ... // other functions we need, e.g. virtual dtor
};

class NormalLevel: public Level {
  public:
    Enemy* createEnemy() override {
      // some code to create mostly turtles
    }
};

class Castle: public Level {
  public:
    Enemy* createEnemy() override {
      // some code to create mostly bulltets
    }
};
```

So, thats the Factory Method Patter, fairly simple.

---

**Now, lets move onto another design pattern:**

### Template Method Pattern

Used when we want subclasses to override some aspects of superclass behavior, but other aspects must stay the same.

e.g. There are red and green turtles

Note: subclasses should not change the way a turtle's feet or head are drawn, but can change the drawing of the shell.

Now, Red and Green are subclasses from Turtle

```Plain
class Turtle {
Public:
  void draw() {
    drawHead();
    drawShell;
    drawFeet;
  }

private:
  void drawHead() {...} // dont want to change, so we dont make it virtual
  void drawFeet() {...}
  virtual void drawShell() = 0;
  // we want red and green to define their own? we make it virtual
  // yes, virtual methods can be be private
};

class RedTurtle: public Turtle {
  void drawShell() override { /* draw red shell */ }
};

class GreenTurtle: public Turtle {
  void drawShell() override { /* draw green shell */ }
};
```

Extension: Non-Virtual Interface (NVI) idiom:

A public virtual method is really two things:

- An interface to the client (public)
    - indicates a promise of provided behavior, class invariants, pie/post methods
- An interface to subclasses (virtual)
    - A "hook" for subclasses to insert specialized behavior

However, it is really hard to separate these two ideas if they're wrapped up in a same function declaration.

- What if you want to later separate the customizable behavior into two methods, with some non-customizable steps in-between? Without changing Public interface
- Your base class is making promises it can't keep, public interface may promise specific behaviors, but derived classes can change that by overriding it.

_**So, the NVF Idiom says:**_

1. All public methods should be non-virtual
2. All virtual methods should be private, or at the least, protected
3. Except the destructor

e.g

```Plain
class DigialMedia {
public:
  virtual void PLay() = 0;
  virtual ~DigitalMedia();
};

// translating this to NVI

class DigitalMedia {
public:
  void play {
    // could have "check DRM();" here
    doPlay();
    // showCoverArt();
  }
  virtual ~DigitalMedia();
private:
  virtual void doPlay() = 0;
  virtual void showCoverArt() = 0;
  vpd checkDRM() {...;}
};
```

Now, we we need exert extra control over play, we can do it:

- e.g. add a `DRM` check before calling `doPLay`
- we can add more "hooks" by additional virtual functions, (e.g. `showCoverArt();`)

All of this without changing public interface. It is much easier to take this kind of control over our virtual methods from the beginning rather then to try and take it back later after much code has been written.

The _**NVI idiom**_ is effectively an extension of the template method pattern, saying **ALL** virtual methods should be in a non-virtual wrapper, essentially no downside as if the wrapper only calls that function, any good compiler will optimize the extra function call.

---

### STL Maps: for creating dictionaries

Example: Map Strings to Ints

```Plain
\#include <map>

using namespace std;

map<String, int> M; // first one is key, secod one is value
M["abc"] = 1;
M["def"] = 4;

cout << M["abc"] << endl; // 1
cout << M["ghi"] << endl; // key will be added if not found, so, initialized (for ints, 0)

// note:
// int n;    // this is uninitialized
// int n{};  // n is default zero-initialized

M.erase("abc");
if (M.count("def")); // will be 0 if not found, map can only store one of each key

// can also iterate through a map
for (auto &p: n) { // suited key order if map
  cout << p.first << " " << p.second << endl;
}

// here, p's type is std::pair<string, int> & (pairs are defined in <utility>
```