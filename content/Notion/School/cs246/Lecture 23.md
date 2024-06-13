# Lecture 23

```Plain
class C{...};
unique_ptr<c> p{new C{...}};
unique_ptr<c> q = p; // error!
```

Class `unqiue_ptr<T>` has no copy operations, its meant to be unique, so a shallow copy of the ptr is disallowed. They can be **from** since that signifies the transfer of ownership.

```Plain
unique_ptr<c> a{new c{...}};
unique_ptr<c> b{make_unique<c> (*a)};
// to copy construct, don't just copy unique pointer
// make your own unique pointer, then assign
```

```Plain
int *p = new int{5};
if (...) {
  unique_ptr<int> q{p};
  ...
 }
*p = 10; // dangling
// should only allocate resources if we are initaizlaing object
// that handles resources. Violated here

// q is destroyed when end of scope, so deallocated 5, so,
// *p is memory error
```

If, however, there is **true** shared ownership, that is, any of several pointers may need to free the data, then you can use **standard share pointers **

### Shared Pointers

```Plain
{ // auto makes p1 a shared_ptr<myClass>
  auto p1 = std::make_shared<myClass>();
  // allocates space for myClass object, p1 points at it
  if (...) {
    auto p2 = p1; // copy construction
  } // p2 is popped of the stack, but, doesn't free the memory
    // it points at, since it is shared
}
// now, p1 is popped of the stack and it does free the memory
```

`shared_ptr`s maintain a reference count, a count of all `shared_ptrs` pointing at the same object. The memory is freed when the reference count reaches 0.

Consider the following:

```Plain
int *p = new int{5};
shared_ptr<int> sp1{p};
if (...) {
  shared_ptr<int> sp2{p};
  }
}
// sp2 doesn't know it is a shared pointer to p, if not copied
// it believes it is the first one

// so, sp2 goes out of scope, count becomes 0, frees data
// then sp1 tries to free, memeory error

// NEED TO SHALLOW COPY
```

Use the pointer type that accurately reflects the ownership role. Dramatically fewer opportunities for leaks this way.

There are **three** levels of exception safety for some function `f`.

1. **Basic guarantee**: If this function throws or propagates an exception, the program will be in a valid but unspecified state. Valid means that nothing is leaked, and class invariants are maintained. Unspecified means undefined.
2. **Strong guarantee**: If this function throws or propagates an exception, the program will be as if the function was never called.
3. **No throw guarantee**: This function will never throw an exception, and will always complete its task

```Plain
class A {...}; // A::g offers strong guarantee
class B {...}; // B::h offers strong guarantee
CLASS C {
  A a;
  B b;
  public:
  void f() {
    a.g();
    b.h();
  }
};
```

From the above code, we can ask: "Is `C::f` exception safe?

1. If `a.g()` throws, nothing happened yet, so ok
2. If `b.h()` throws, the effects of `a.g()` must be undone to offer the strong guarantee. Very hard or impossible if `a.g()` has non-local side-effects (e.g. printing to output).

Now, assuming there are no non-local side-affects, lets make this exception safe:

```Plain
// Assuming no non-local side-affects, we can use
// copy-swap idiom
class C {
  A a;
  B b;
  public:
  void f() {
    A atemp[a};
    B btemp{b};

    atemp.g();
    btemp.h();
    // if either of these throw, to the outside observer,
    // state has not changed

    a = atemp;
    b = btemp;
       }
};
// however, we have the same problem, if b = btemp throws
// but a = atemp did not
```

Because the copy assignment could throw, we don't yet have exception safety. It would be better if we could guarantee that the "swap" part was a no-throw operation - a non-throwing swap operation is at the heart of writing exception safe code.

**Key observation:** Copying pointers can't throw.

```Plain
class CImpl {
  A a;
  B b;
};

class C {
  unique_ptr<CImpl> pImpl;
  public:
  void f() {
    auto temp = make_unique<CImpl> (*pImpl);
    temp->a.g();
    temp->b.h();

    std::swap(pImpl, temp); // no thro, just swaping pointers
  }
};
```

On the other hand, if either `A::g` or `B::h` offer no exception safety guarantee, then in general, neither can `C::f`. Similarly if they have non-local side fix, it is much harder to offer a guarantee for `C::f`.

---

### Exception Safety and STL Vectors

**Vectors**:

- Encapsulate a heal-allocated array
- Follow **RAII** - with a stack allocated vector goes out of scope, the internal heap allocated array is freed.

Quick review:

```Plain
void f() {
  vector<myClass> v;
  ...
 }
// v goes out of scope, array is freed, so the myClass objects
// stored call their destructor
```

**But!**

```Plain
void g() {
  vector<myClass *>v;
  v.emplace_back(new myClass());
  ...
}

// here, array is freed, pointers don't have dtor, leaked
```

So, it you need to free these pointers, would need to do it yourself.

Alternatively, we could make a `vector` of `unique_ptr`s.

```Plain
void h() {
  vector<unique_ptr<myClass>> v;
}

// array is freed, unique_ptr's destructor frees the myClass
// objects
```

Now, consider how `vector<T>::emplace_back` (or push_back) works.

- Offer the strong guarantee
- If the array is full, (i.e size = capacity)
    - Doubling, new larger array allocated, copied in
        - Adds the new element
        - Copies the objects from the old (copy ctor)
            - We don't use move to ensure exception guarantee (say a move operation fails)
            - If the copy construction throws, we destroy new larger array, old array still intact, strong guarantee
        - delete old array
- * But**, copying is expensive if old array is just going to be thrown away. Furthermore, how can we create a vector of `unique_ptr`s if they can't be copied?? wUT..

Wouldn't moving the object from old array to new array be more efficient?

- Allocate new larger array
- Place new object in it
- Move the objects over (move ctor)
- Delete old array

However, `emplace_back()` offers the strong guarantee. The problem, if we move, if move ctor throws, then `vector<T>::emplace_back` can no longer offer the strong guarantee, since old array is no longer intact. But, `emplace_back` promises the strong guarantee, so how does it work? We know it can use move ctor as we can have vector of `unique_ptr`, and that class does not offer copy ctor, unique.

Therefore, if the move constructor offer the no throw guarantee, `emplace_back` will use the move ctor, otherwise, it will use the copy ctor, may be slower.

This is why we place a new object into the new array first.

Your move operations should provide the no-throw guarantee if possible, and you should indicate that they do.

```Plain
class MyClass {
  public:
  MyClass(MyClass &&other) noexcept {...}
  // noexcept keyword promises compiler, will not throw
  MyClass &operator=(MyClass &&other) noexcept {...}
  ...
};
```

If you know that a function will not throw or propagate an exception, declare it `noexcept`, facilitates optimization. At a minimum, your classes swap and move operations should be non-throwing.