Sometimes, later one, you will see instances where “?” is used in the initialization of objects. Here is what it means:

```C++
condition ? result_if_true : result_if_false
```

Now, we move on with the lecture:

```C++
struct Student {
  int assignments, mt, final;
  float grades() {...};
  Student (int assignments, int mt, int final);
}

Student::Student(int assignments, int mt, int final) {
  this->assignments = assignments;
  this->mt = mt;
  this-> final = final;

  if (this->assignments < 0) {
    this->assignments = 0;
  } else if (this->assignments > 100) {
    this->assignments = 100;
  }
}

Student s{60,70,80};
// BETTER!
```

- If a constructor has been defined, the numbers above are passed as arguments to the constructor
- If no constructor has been defined, then these initialize the individual fields of student
- Alternatively:

```C++
Student abdur = Student{60,70,80}; // does the samr thing
```

**A note on uniform initialization**

- Sometimes, variables are initialized with `=`

```C++
int x = 5;
string s = "hello";
```

We may also initialize by `()`, especially when invoking constructors

```C++
int x(5);
string s("helo");
Student michelle(60,70,80);
```

- The rules about when we may use `=` vs `()` are somewhat arbitrary.
- `C++`now has uniform initialization syntax, using `{}`, which is meant to be in nearly all initializations

```C++
int x{5};
string allan{"allan"};
Student michelle{"michelle"};
```

Try to get in the habit of using `{}`for initialization, it is the preferred modern `c++`style.

---

**For heap allocation**

```C++
Student *pjane = new Student{90,90,90};
```

**Advantages of constructors**

- "Sanity" checks, logic checks, default parameters, logic other than simple filed initialization

For example:

```C++
struct Student {
  ...
  Student(int assignments = 0; int nt = 0; int final = 0);
  ...
}

// now in the implementation
Student::Student(int assignments, it mt, int final) {  ...
}

// so, we need only put the defaults in the declaration, not in implementation
```

  

so, the following would be able to defined:

```C++
Student Mary{70,80}; // 70,80,0
Student newkid; // 0,0,0
```

**Note, the 0 parameter constructor above is called a** `**default constructor**`

The default constructor default constructs all fields which are objects. The built in default constructor goes away as soon as you provide any default constructor:

```C++
struct Vec {
  int x,y;
  Vec(int x, int y) {
    this->x = x;
    this->y = y;
  }
};

// so

Vec v{1,2}; // this is ok; in the same order fields are declared, so x first, then y
Vec w; // error!
```

So, the default constructor is just going like `Class allan;`, not passing it any arguments or passing it empty brackets. This is why `Vec v;`, in this, the default constructor does nothing. There are no objects inside of the definition of `Vec`.

```C++
struct Vec {
  int x,y;
  Vec (int x, int y) {
    this->x = x;
    this-> y = y;
  }
};

// so now, Vec has NO defualt constructor

Struct Basis {
  vec v1 v2;
};

Basis b; // so this won't exist, compilation error.
```

So, now, this basis's default constructor will go and constructs the fields which are objects, which are the two `vec` objects. However, the `vec` class has no default constructor, hence causing an error.

---

**What if a structure contains constants or references**

```C++
Struct MyStruct {
  const int myConst;
  int &myRef;
}; // this also causes you to lose def. consturcotr
```

**Often, you will lose the default constructor as we write our own constructors, just write a new default constructors**

```C++
int 2;
struct MyStruct {
  const in myConst = 5;
  int &myRef = 2;
};
```

But, should every `MyStruct` object need to have the same value of `myConst` and `myRef`?

**For example:**

```C++
struct Student{
 const int id; // does not change, different for every student
```

Where to initialize? Constructor body? Too late. By then, fields must be fully constructed by the time the constructor body runs.

- *What happens when an object is created? **

1. Space is allocated
2. Fields are initialized
3. Constructor body runs

So, we need to state our fields before constructor body is run. So, how do we specify how to initialize fields in `step 2`? We use `Member-initialization List` (MIL).

---

**Member-initialization List**

```C++
struct Student {
  const int id;
  int assignments, mt, final;
  Student (int id, int asignments, int mt, int final):
    id{id}, assignments{assignments}, mt{mt}, final{final} {
      // nothing in here
    }
};
```

- As seen above, the `:` is the Member-initialization List. Below it, we initialize every field of the current object (Student, id, assignments, mt, final) which are the things outside of `{}` with the parameter passed to the constructor, which in this case, are also called (id, assignments, mt, final) and are inside the `{}`.
- *NOTES **
- You can (and should) initialize any `field` in the `MIL`, not just `consts` and `references`
- Fields listed in the MIL are initialized in the order they are declared in the class, regardless of the order they show up in the `MIL` (you'll get a warning if these differ)
- The `MIL` is more efficient than assigning values in the constructor body, for fields that are objects, if they are not listed in the `MIL`, then they're default constructed; if you then assign them in the body, they are then initialized then assigned which is wasteful

The `MIL` is more efficient than assigning values in the body of the constructor, for fields that are objects, if they are not listed in the `MIL`, then they're default-constructed; if you then assigned them in the bodt, they are then initialized then assigned which is wasteful

**EMBRACE THE MIL**

What if the field is initialized both in the class and in the `MIL`?

```C++
struct Vec {
  int x = 0, y =0;
  Vec(int x) : x{x} {
  }
};
```

In this case, the `MIL` takes precedence, so Vec v{5};

Now consider:

```C++
Student Mahmoud{60,70,80};
Student Caroline = Mahmoud;
```

how does `caroline`s initialization occur? Via the copy constructor, which is invoked when an object is constructed with another object of the same type

**Note every-class**

- defualt consturctor
- a copy of constructor (just copy intilaizes all fields)
- a copt assignment operator
- a destructor
- a move consturcor
- a move assignment