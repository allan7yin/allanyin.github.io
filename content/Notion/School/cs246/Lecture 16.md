# Lecture 16

**Relationship**

```Plain
// base class/ super class

class Book {
  string title, author;
  int numPages;
  Public:
    Book(...);
    ...
 };

class Text {
  string title, author;
  int numPages;
  string topic;
  Public:
    Text(...);
    ...
 };

// For comic books, you want to track the hero

class Comic {
  string title, author;
  int numPages;
  string hero;
  public:
    Comic(...);
    ...
 };
```

Now, the above is ok, it's fine to define three separate classes for the entities listed above. But, such a practice does not capture the relationship among `Comics`,`Texts`, and `Books`. Moreover, how do we create an array or other collection that contains a mixture of these.

To model such relations, we use `inheritance`.

```Plain
// derived class (or subclass)
class Text: public Book {
  string topic;
  Public:
    Text(...);
     ...
};

class Comic: public Book {
  string hero;
  public:
    Comic(...);
    ...
};
```

Derived classes inherit fields and methods from their base class. So, `Text` and `Comic` get `title`, `author`, and `numPages` fields from `Book`.

Any method that can be called on `Book`can be called on a `Comic` or `Text` because they are `Book`s.

But, who can access the `title`, `author`, and `numPages` fields? Can subclasses see them? **NO**, not even subclasses can see/access private methods of the superclass.

So, then, how do we initialize a `Text` object?

```Plain
// Base class/superclass

class Book {
  string title, author;
  int numPages;
  Public:
    Book(string title, string author, int numPages): title{title}, .... {}
};


class Comic: public Book {
  string hero;
  public:
  Comic(string title, string author, int numPages, string hero): Book{title, author, numPages}, hero{hero} {}
};
```

The above is necessary for two reasons:

1. `title`, `author`, and `numPages` are not accessible by `Comic`
2. Again, when an object is constructed:
    - Space is allocated
    - Superclass components are initialized
    - fields are initialized
    - constructor body runs

`(2)` will not work if we don't specify it because if unspecified, the compiler tries to default construct our superclass and `Book`has not default constructor. There are many good reasons to keep superclass fields hidden from derived classes (encapsulation - less places to look for bugs, regarding those fields). But if you want to give a subclass access to certain fields, you can use the `protected` visibility.

```Plain
class Book {
  protected: // accessible to Book and its derived classes
    string title, author;
    int numPages;
  public:
    Book(string title, string author, int numPages): title{title}, author{author}, numPages{numPages} {}
};

class Text: public Book {
  string topic;
  public:
    Text(...);
    ...
    void addAuthor(const string &a) {
      author += a;
    }
};
```

We cannot initialize `Book` components in `MIL` because superclass components are already initialized before subclasses components.

Better for encapsulation to keep fields private and provide protected getters/setters (that way those methods can maintain invariants).

```Plain
class Book {
  string title, author;
  int numpages;
  protected:
    void setAuthor(const string &a) {author = a;}
    string getAuthor() {return author;}
  public:
    ...
};

void Text::setAuthor(const string &a) {
  setAuthor(getAuthor() + a);
}
```

This relationship among `Text`, `Comic`, and `Book`, is called an "is-a" relationship. A `Text` is a `Book`, and a `Comic` is a `Book`.

The "is-a" relationship is implemented via public inheritance.

---

Now, consider we want to write a method `isHeavy`. When is a `Book` heavy?

- For ordinary books, 'Heavy' means > 200 pages
- For textbooks, 'Heavy' means > 500 pages
- For comics, 'Heavy' means > 50 pages

```Plain
class Book {
  ...
  int numPages;
  public:
  ...
  bool isHeavy() {return numPages > 200;}
};

class Text: public Book {
  ...
  public:
  bool isHeavy() {return numPages > 500;} // assume numPages is protected
};

class Comic: public Book {
  ...
  public:
  bool isHeavy() {return numPages > 50;} // assume numPages is protected
};

Book b{"A small book", "Dave", 50};
Comic c{"comic", "someone", 60, "Recycling"};

cout << b.isHeavy() << c.isHeavy() << endl; // False and True

// Because public inheritance is an 'is-a' relationship, we can write
Book b = Comic{"Big comic", ..., 75, ...};

// however, is b heavy? NO, b is a book, and so, Book::isHeavy() runs
```

Here, the compiler will attempt to fit a `Comic` where there is only space for a `Book` object - effectively, the `Comic` is 'sliced'. The `'hero'` field is chopped off and the `Comic` is coerced into a `Book`.

But, if we access through pointers, no slicing is required.

```Plain
Comic c{..., 60, ...};
Book *pb = &c;
Comic *pc = &c;

pc->isHeavy() // True
pb->isHeavy() // False

// Still false when called through a Book pointer even though points at a Comic

int main() {
  Book *pb;
  int n;
  cin >> n;
  if (n < 0) {pb = new Book{..., 60, ...};}
  else {pb = new Comic{..., 60, ...};}
  pb->isHeavy();
}
```

The compiler doesn't know if `pb` points at a `Book` or `Text` or `Comic`, it only knows `pb`'s `static` type is that of `Book` pointer. So it calls `Book::isHeavy`. When pointed at by a base class pointer/reference, these objects behave as their base class.

How do we make a Comic act like a Comic even when pointed at by a base class pointer?