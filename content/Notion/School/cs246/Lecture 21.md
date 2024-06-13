# Lecture 21

Last class: Maps

### Design Pattern: Visitor Pattern

For implementing double dispatch. Virtual methods are chosen based on the run-time type of an object on which they're called. What if you want to choose a method based on two objects?

**Example: Striking enemies with weapons**

We want something like

```Plain
virtual void(Enemy, weapon)::strike)
```

. But, that is not possible.

If we write `virtual void Enemy::strike(weapon &w)`, then the method is chosen based on the enemy type but not the weapon type.

If we write `virtual void weapon::strike(Enemy &e)`, then we choose based on weapon type but not enemy type

The trick to getting double dispatch is to combine overloading and overriding.

```Plain
class Enemy {
public:
  virtual void beStruckBy(Weapon &w) = 0;
};

class Bullet: public Enemey {
public:
  void beStruckBy(Weapon &w) override{
    w.Strike(*this);
  }
};

class Turtle: public Enemy {
public:
  void beStruckBy(Weapon &w) override{
    w.Strike(*this);
  }
};

// these twop classes are very different. Although similar code, *this passes different type

class Weapon {
public:
  virtual void Strike(Turtle &) = 0;
  virtual void Strike(Bullet &) = 0;
};

class Stick: public Weapon {
public:
  void Strike(Turtle &t) {/* strike turtle with stick*/ }
  void Strike(Bullet &b) {/* strike bullet with stick*/ }
};

// class Rock is similar as stick
int main() {
  Enemy *e = ...; // something, dont need to know what it is
  Weapon *w = ....; // something, dont need to know what it is
  e->beStruckBy(* w);
  // what happens here?

```

- Virtual dispatch for `beStruckBy` occurs, calling `(Turtle or Bullet)::beStruckBy`
- The appropriate overwritten method calls `w.strike` on itself, the argument reference type is then a `Bullet` or a `Turtle` based on which method was chosen above
- Then, the appropriate `strike` method is chosen based on virtual dispatch.
- Our `stick` or our `rock`, hits the `turtle` or `bullet` appropriately.

**However, this design pattern has issues:**

- We want re-usability, abstraction, and extendability

Visitor pattern can be used to add functionality to existing classes without changing or recompiling them, so long as they offer the the "visit" interface.

**Example:** Adding a visitor to the `Book` hierarchy

```Plain
class Book { // Book should be an "Enemy"
public:
  ...// someting here
  virtual void accept(BookVisitor &) { // similar to "beStruckBy"
    v.visit(*this);
  }
};

class Comic : public Book {
public;
  ...// something here
  void accept(BookVisitor &v) {
    v.visit(*this);
  }
};

// Text class is similar
// Now, we can define an abstract visitor
class BookVisitor { // like "wepaon"
public:
  virtual void visit(Book &b) = 0; // similar to "strike"
  virtual void visit(Comic &c) = 0;
  virtual void visit(Text &t) = 0;
};
```

Now, let's write an **Application** of our `BookVisitor`class.

**e.g.** How many of each type of `Book`, we have:

- group `Books` by author
- group `Text`s by topic
- group `Comics` by hero

```Plain
struct Catalogue: public BookVisitor {
  Map<string, int> theCatalogue;
  void visit(Book &b) { ++theCataglogue[b.getAuthor()];}
  void visit(Comic &c) { ++theCataglogue[c.getHero()];}
  void visit(Text &t) { ++theCataglogue[t.getTopic()];}
};
```

So, the `Book` and `Catalogue` code above wont compile. Why? `book.h` includes `Bookvisitor.h`, which includes `text.h`, `text.h` includes `book.h` - a circular inclusion.

Because of the header guard, `text.h` doesn't actually get a copy of `book.h`, so, the compiler doesn't know what a `Book` is when we define `Text` as a subclass.

Our `text.h` absoultely needs `Book.h`. So, this raises the question, are all of these `\#include`s necessary?

---

### Compilation Dependencies

When does a compilation dependency exist, i.e., when does a file **REALLY** need to include another?

Consider:

```Plain
// a.h
class A{...};

// b.h
class B: public A {
  // inherits from A, need to know evertything about A
  // so here, we include A.h

  // how big is a B object? Well, we need to first know how
  // big an A object is
};

// c.h
class C {
  A myA;
  // size of C object has A object, so, for memory to be
  // allocated for a C object, needs to know A size

  // so here, we include A.h
};

// d.h
class D {
  A *myAp;
  // here, we dont need to include
  // this a pointer, always same size, dont need to know the
  // size of A

  // just needs to know existence of A, forward declare will suffice
};

// e.h
class E {
  A f(A x); // x is of type A

  // we're just declaring the function that takes an A
  // all we need to know A exists, dont need specifics of A

  // dont need A.h
};

// f.h
class F {
  A f(A x) {
    x.SomeMethod();
  }

  // here, similar to E, but we're actually writing the
  // function that uses A, so we need to include

  // need to know A has a function has a method called
  // SomeMethod

  // so, we need to inlclude A.h here
};
```

In general, do not introduce compilation dependencies where they don't actually exist. Forward declare when possible, and include only when necessary.

Now, the implementation files will likely have true dependencies, but that's OK.

---

Consider the Xwindow class:

```Plain
class Xwindow {
  Display *d;
  Window w;
  int s;
  GC gc;
  unsinged long collows[10];
};

// Do we know what the private things mean? If we're using it
// who cares! Point of class is to not need to know, just use
// public interface
```

What if we add, or change, a private member? If so, all client code must recompile. This seems unnecessary. The clients shouldn't care about private.