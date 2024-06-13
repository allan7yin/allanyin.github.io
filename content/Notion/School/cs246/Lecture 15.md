# Lecture 15

- *Static members of classes **

```Plain
struct Student {
  ...
  mutable it numMethodCalls = 0;
  float grade() const {
    numMethodCalls++;
    return ...
    }

```

The `countMethodCalls` is for a particular object. It is hard to keep track of all method calls across all student objects. What if we wanted to keep track of something entirely different, something that doesn't make sense to be specific to an object. For example, what if we want to keep track of how many Student objects are created?

What we want is a variable that is associated with the class itself, not a particular instance of the class. This is why we have `static`members.

```Plain
struct Student {
  int assns, mt, final;
  static int numInstances;
  Student(int assns, int mt, int final) : ... {
    numInstances++;
  }
};
```

In order to initialize the `static`variable, we do so outside of the class:

```Plain
int Student::numInstances = 0;
```

Static fields must be initialized outside the class. Similarly, we can also have `static member functions`. Static member functions don't depend on a specific instance for their computation. So, a static member functions can only access the static members. So, static member functions do not have a `this`pointer. Thus, as said above, they can only access static fields and members. They cannot call `non-static`methods.

For example:

```Plain
struct Student {
  ...
  static void howMany() {
    cout << numInstances << endl;
  }
};

Student Billy{60,70,80};
Student Yei{50,80,80};

Student::howMnay(); // 12
Yei.howMany(); // 12
```

```Plain
struct Foo{
  static int x;
  void doSomething() const{
    x++;
    cout << hello << endl;
  }
};

int Food:x = 0;
int main() {
  Foo f;
  f.doSomething(); // so this is ok
  cout << foo::x << endl; // find since public, if private, wont work
}
```

So, `f.doSomething()`works. The `const` in the `void soSomething()` just says we are not going to change the object, incrementing x changes the `class`, but not the `object`.

---

Now, we look at the `input` and `output`operators as members.

```Plain
class Vec{
  int x,y;
Public:
  ...
  int getX() const {return x;} // method is called an accessor or getter
  int setY(int z) { y = z;} // method is called a mutator or setter
};
```

Providing access to private fields can be done trough accessor/mutator methods when appropriate. These methods can include code to check/maintain invariants.

- *What about input/output operators? **

You may initially think that overloading it in the class may be the way to go. However, it makes things very uncomfortable and weird. So, instead, we can use `friend` in classes:

```Plain
class Vec{
  ...
  friend std::ostream &operator<<(std::ostream &out, vec &v);
};

// then, outside of the class

ostream &operator<<(ostream &out , vec &v) {
  out << v.x << ", " << v.y;
  return out;
}

// remember that for operator overloading like this, we need to
// return the output stream reference.

// Also, as a friend function, you are not a member function,
// you are simply an external function that has access to the
// class parameters
```

If you already have all the getters/setters, you need to write `I/O operators`, then you don't need to declare it as a friend, but don't add all getters/setters **JUST** for the `I/O operators` because then everyone can access them, not just the `I/O` stream.

---

### System Modeling

This is a way to visualize the structure of a system (abstractions and the relationships among them) to aid in design and implementation.

One popular standard is **UML Class diagrams**

Modeling a class in UML. Here is how to draw a class in UML and put it in a box.

Class Name  
Fields (optional)  
Methods (optional)  

Then, we put each one in sub-boxes and outside has a box. Check photos for reference. Will upload below:

---

### Relationships: composition of classes

```Plain
class Vec{
  int x,y;
public:
  Vec(int x, int y) : x{x}, y{y} {}
};
```

Suppose that two vectors define a basis:

```Plain
class Basis{
  vec v1, v2;
  ...
};

Basis b; // ERROR,
```

The built-in default constructor for `Basis`tries to default construct `v1` and `v2`, no such `Vec::Vec` constructor exists so we ca write our own that explicitly initializes them:

```Plain
class Basis {
  Vec v1, v2;
public:
  Basis(): v1{1,0}, v2{0,1} {}
  // invikes consturcotrs on v1 and v2 - must be done in
  // the MIL - remember by time constructor body runs fileds are initialized
```

Embedding objects (e.g. v1 and v2) within anther (e.g. `b`) is a composition relationship.

Relationship: a `basis`object "owns-a" `Vec`object (in fact, it owns two).

If `A`"owns-a" `B`, then typically:

- `B`has no identity outside of A (no independent existence)
- If `A`is destroyed, then `B`is also destroyed
- If `A`is copied, then `B`is copied (e.g. deep copy)

An example of this are `Node` and `List`, from last class. `List`owned the head node and each `Node`object owned a pointer to another `Node`.

Composition is typically implemented through embedding objects within another, or through a pointer of which you have ownership.

So how do we model this in **UML**?

Filled in diamond attched means it owns another class. So, to show `Basis` owns `Node`, we see the below:

(insert photo here)

Composition is a solid filled in diamond

- The 2 says "eacb `basis`owns exactly 2 `vec`
- The 1 says "each `vec` is owed by exactly one `basis`

---

### Relationship: Aggregation

Students in a course, courses have students, but, students have an independent existence outside of the course.

This is a "has-a" (aggregation) relationship. So:

- If a`A` "has-a" `B`, then typically
    - `B` exists apart from its association from `A`
    - If `A` is destroyed, `B`lives on
    - if `A` is copied, `B` is not (shallow copy). So copies of `A` share the same `B`.

**Example:** parts in a catalogue, so if we rip up the catalogue, the parts don't "cease to exist"

- * Modeling this:**

(insert photo here)

Here, the diamond is not filled in. /* Writing the asterick means any number. Writing 10 ... star, means 10 to any number of parts.

**Typical Implementation** of aggregation relationship:

- A pointer or a reference, particularly one you don't own
- Is through a non-owning pointer or reference

For example:

```Plain
class Catalogue {
  Part &parts[500];
  ...
public:
  ~Catalogue() {// dont delete the parts}
};
```

- *Relationship: Specialization **

Suppose you want to keep track of your collection of books.

```Plain
class Book {
  std::string title, author;
  int length;
public:
    Book(...);
};
```

For textbooks, also want to know the topic:

```Plain
class Text {
  std::string title,author;
  int length;
  std::string topic;
```