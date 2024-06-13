# Lecture 19

```Plain
class Book {
  public:
    // defines copy/move constructor and assignment operators
};

class Text: public Book {
  string topic;
  public:
    // don't define copy/move constructors
};

Text t{"Algorithms", "CLRS", 550 "CS"};
Text t2 = t;

// no copy constructor in Text, what happens?
```

This copy initialization calls `Book` constructor and then goes filed-by-field copy initializing (the built-in behavior) the same is true for other compiler provided methods.

To write our own operations:

```Plain
Text::Text(const Text &other): Book{other}, topic{other.topic} {}
Text::Text(Text &&other): Book{std::move(other)}, topic{std::move(other.topic)} {}
```

**Note:**  
Even though the object  
`other` refers is an `rvalue`, `other` itself is a named parameter of a function (so it has a lifeline) so `other` itself is an `lvalue`. The `<utility>` header includes the function `std::move` which forces an `lvalue` to be treated as an `rvalue`, so we can all the move operations on it.

```Plain
Text &operator=(const Text &other) {
  Book::operator=(other);
  topic = other.topic;
  return *this;
}

Text &operator=(Book &&other) {
  Book::operator=(std::move(other));
  topic = std::move(other.topic);
  return *this;
}
```

The behavior we have written for these functions is the same as the compiler-provided specialize as need for classes that require it.

- *Now consider: **

```Plain
Text t1{...}, t2{...};
Book *pb1 = &t1, *pb2 = &t2;
*pb1 = *pb2 // what happens? Book assignment operator
  // runs because method is not virtual
```

The result is only the Book components assigned so this is partial assignment.

- **So, how do we fix this? ***

```Plain
class Book {
  ....
  public:
  virtual Book &operator=(const Book &other) ... // as before
};

class Text: public Book {
  ....
  public:
  Text &operator=(const Book &other) override {
    Book::operator=other;
    topic = other.topic; // illegal, other is a Book,
    // can't do this
    return *this;
  }
};
```

Note: `Text::operator=` is permitted to return (by reference) a subtype object and still be valid override, **BUT** the parameter types **MUST** be the same, or it is not an override, so by 'is-a' principle, if a `Book` can be assigned by another `Book`, then a `Text` can be assigned to a `Book`.

```Plain
Text t{...};
Book b{...};
Text *pt = &t;
*pt = b; // calls operator= with virtual dispatch,
// uses a Book to assign to a text, would be BAD
```

Also, if we had something like:

```Plain
Comic c{...} // some implementation
t = c;
// this uses a comic object to intialize a text object, bad!
```

Before the `operator=` was virtual, we had partial assignment through base class pointers, if it is virtual, we get this problem of mixed assignment.

Assignment to base class pointers/references doesn't make sense - we need to know the two objects we are assigning are the same type. This defeats the purpose of working polymorphically.

To prevent this, we disallow assignment through base class pointers by making the assignment operator private or protected in the base class - can stay public and non-virtual in derived classes.

```Plain
Text t{...} // some implementation
Text q{...}
t = q; // allowed
Book *p = &t;
Book *z = &q;

*p = *z; // illegal

// Now, it is also illegal to do:
Book b{...}
Book d{...}
b = d; // which is not what we want, so, we ideally want to
// have an ABSTRACT base class
```

For example, we can do:

```Plain
class AbstractBook {
  string title, author;
  int numPages;
protected:
  AbstractBook &operator(const AbstractBook &other) {...}
public:
  AbstractBook(...);
  virtual ~AbstractBook() = 0;
};

// implementation for ~AsbtractBook somehwere
class NormalBook: public AbstractBook {
public:
  NormalBook{...}
  ~NormalBook() {}
  NormalBook &operator=(const NormalBook &other) {
    AbstractBook::operator=(other);
    return *this;
  }
};

// this sort of design prevents both partial and mixed
// assignment
vector<int> v{1,2,3};
for (int i = 0; i < 10000; i++) {
  cout << v[i] << endl;
}

// the index operator for vectors is unchecked - assumes you are indexing a valid index

vector::at(); // this is a checked version of indexing
for (int i = 0; i < 10000; i++) {
  cout << v.at(i) << endl;
}

```

- Here, at checks if valid index, but what happens if `i` is not a valid index?
- In `C++`, an exception is raised.

---

### Exception Handling

An exception is some value raised by function that ends execution of that function and passes error along to the caller.

For example, `vector.at()` can detect that an error has occurred, but does not know what to do with it. `vector.at()` specifically raises a `std::out_of_range` object when a bad access is made. `std::out_of_range`is an exception class from the header `\#include <exception>`. So, what do we do about exceptions.

```Plain
void f() {
  vetor v{0.2};
  v.at(10000);
}

void g() {
  f();
}

int main() {
  g();
}
```

Here, `main` will call g, which calls `f`, which then calls `vector.at()` which raises an exception. By default, the program stops executing.

More specifically, the `f()` propogates the error to `g()`, which does the same thing, so the error reaches `main`. If an error reaches all the way to `main`, and it is not handled in `main`, the program terminates.

To prevent this, we can catch errors:

```Plain
int main() {
  try {
    g();
  } catch (std::out_of_range e) { // e is exception object
    cout << "got error" << e.what() << endl;
  }
}
```

If an exception raising code is wrapped in a `try/catch` block, it can handle the exception if it has a matching handler.

What if `g()` catches the exception, handle some behaviour, and raises another exception?

```Plain
class SomeError{};

void g() {
  try {
    f();
  } catch (std::out_of_range) {
    ...
    throw SomeError{};
  }
}

int main() {
  try {
    g();
  } catch (SomeError e) {
    cout << "got error" << endl;
  }
}
```

What if `g()` wanted to raise the same error?

```Plain
void g() {
  try {
    f();
  } catch (std::out_of_range e) {
    throw;
  }
}
```

Why do we just write `throw` instead of `throw e`.