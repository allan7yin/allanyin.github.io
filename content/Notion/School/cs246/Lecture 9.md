What if we want a module to provide a global variable

```C++
//abc.h

int globalNum;  // wrong, this is a definition and a declaration

// probelm: every file that includes abc.h gets its own copy. of thus global variable, it is then defined multiple times, and the program wont link

// solution: put the definition of the variable in the .cc file so it gets only compiled once and only declare it in the header
```

So,

```C++
//abc.c

int globalNum; // def'n (assign value if wanted)
```

```C++
//abc.h

extern int globalNum; // only declares function, no definition
```

### What is linking?

- The linker combines compiled object files into a single executable - in order to do so it not only combines the files but resolved external symbols
- Symbols that are declared not defined in a .cc file must be resolved by the linker
    - e.g. In our `vec`file, we declared an overload addition operator between two `vec`'s.

```C++
///vec.h

struct vec {
  int x.y;
}

vec operator+(const vec &v1, const vec %v2);
// we only declared, definition was in .cc file
```

And yet, [main.cc](http://main.cc/)

```C++
\#include "vec.h"

int main() {
  vec v{1,2};
  v = v+v;
}
```

- Called this function, and was compiled, but it only knows about the function declaration
    - That's because the compiler sees this function has been declared, generates place holder info in the object file to say it wants to call it, the linker then finds it in the other object file and updates the location of it in the compiler's placeholder code

Suppose we wanted to create a linear algebra module.

```C++
// linalg.h

\#include "vec.h"
```

```C++
//linalg.cc

\#include "linalg.h"
\#include "vec.h"
```

```C++
//main.cc

\#include "linalg.h"
\#include "vec.h"
```

This doesn't compile, `main.cc`and `linalg.cc` both include `vec.h` and `linalg.h`, but `linalg.h` includes `vec.h`. So, these two `.cc`files get two copies of. `vec.h`, and thus two definitions of `struct vec`. We cannot define something twice, so this is an error.

So, how can we fix this? One file is to remove `vec.h` from `linalg.cc`, but this is not good, since sometimes, it is impossible to tell what each `include` has.

**Solution: Header guard** `**\#include guard**`

```C++
//vec.h
\#ifndef VEC_H
\#define VEC_H

// then contents of vec.h goes here

\#endif
```

- The first time `vec.h`is included, the symbol `VEC_H`is not defined, so the file (including the definition of VEC_H) is included.
- Subsequent includes `VEC.H`is defied, so the `\#ifndef`is false, and the file is suppressed.

---

- *Always: out `\#include` guards in .h files

**DON'T: Put** `**using namespace**` **in** `**.h files**`. It forces files that include your header to have that using directove

- *NEVER EVER include .cc files **

**NEVER EVER compile .h files, their code gets compiled as part of the files that they are included in**

---

### **Classes**

- **Class:** Can put functions inside of `structs`.
- Can make functions that can only be called on an existing piece of data
- Object is an instant of a class.

```C++
//student.h

struct Student {
  int assignments, mt, final;
  float grade();
};
```

```C++
//student.cc

float Student::grade() {
  return assignments*0.4 + mt*0.2 + final*0.4;
}
```

```C++
//client code
\#include "student.h"

int main() {
  Student s{60,70,80};
  cout << s.grade() << endl;
}
```

Function inside of a class is called a member function, or method

Some **OOP ** terminology

- A class is essentially a structure type that can potentially contain functions
    - `Student`above, is a class
- An object is an instance of a class
    - `s` above, is an object, instance of class `Student`
- The functions are called `member functions` or `methods`
- Likewise, the variables in a class are called `member variables` or `fields`
- `members` just refers to both the functions and variables
- `::` is called the scope resolution operator
    - `c::f` means f in the context of `c` `(class or namespace)`, where `::`
        - LHS is a `Class (or nampespace)` rather than an object
- *So, ** inside the function `grade()`, what do the assignments, mt, and final mean?
- Since these fields don't exist until an object is created/instantiated, they refer to the fields of the object that the method is called on.

e.g.

```C++
Student billy{...}
billy.grade(); // uses billy's assignments, mt, and final
```

Formally, methods take a hidden extra parameter which is called `this`, which is a pointer to the object the method wad called on. In above, `this == &billy`. So every member function takes this parameter.

The above is the same as the following:

```C++
float Student::grade() {
 return this->assignments*(0.4) + this->mt*(0.2) + this->final*(0.4);
}
```

Analagous to what we wrote before, the `this->` was implicit. Only need to specify `this->field` if there's another variable in scope with that name.

- *Initializing Objects **

```C++
Student abder{60,70,80}; // ok, but limited
```

Better solution:

- Include a method that initializes called a `constructor`, needs the same name as the `class`.

```C++
// student.h
class Student {
  int assignments, mt, final;
  float grade();
  Student(int assignments, int mt, int final);
};
```

```C++
//student.cc

Student::Student (int assignments, int mt, int final) {
  this->assignments = assignments;
  this->mt = mt;
  this->final = final;
}
```