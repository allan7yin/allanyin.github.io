Recall: The preprocessor transforms your program before the compiler sees it

```C++
\#include<iostream>
```

`include` is a preprocessor directive

```C++
\#define VAR VAL
```

This sets a preprocessor variable to have value, all occurrences of `VAR` in your source file get replaced with `VAL` except occurrences in quoted strings.

```C++
\#define MAX 10
int x[MAX];

// would be transformed into x[10];
```

However, this is not used a lot in `C++`, in fact, almost never used. It is ALWAYS better to just use `const`.

```C++
\#define FLAG
```

- This sets the variable `FLAG`, its value is the empty string
- Defined constants are useful for conditional compilation
    - What this means is hiding code from the complier that you don't necessarily want compiled

For example:

```C++
\#define SECURITYLEVEL 1

# if SECURITYLEVEL == 1
short int
\#elif SECURITYLEVEL == 2
long long int
\#endif
```

Special case:

```C++
\#if 0

...

\#endif
```

`\#if 0` is never true, so all of the inner text is removed before the compiler sees it, works as a heavy duty "comment out".

We can also define preprocessor symbols, via the `command-line` compiler arguments.

Say we have the following incorrect code:

```C++
int main() {
  cout << x << endl;
}
```

Then, to compile it, we:

```Bash
g++ -DX=15 define.cc -i define
./define
15
```

x is now set to `15` through the command-line.

`\#ifdef`

`NAME/\#ifdef NAME`, true if `var/flag` has/has not been defined.

```C++
int main() {
  \#ifdef DEBUG
  cout << "setting x = 1" << endl;

  \#endif
  int x = 1;
  while (x < 100) {
    ++x;
    \#ifdef DEBUG
    cout << "x is now" << endl;
    \#endif
  }
}
```

Compiling this: `g++ loop.cc -o loop`, we see no print statements

Now, with:

```Bash
g++ -DEBUG loop.cc -o
./oop
```

With the above, we see our prints.

---

### Separate compilation

The point of this is to split the program into composable modules, each which provide:

- Interface file:
    - `type` definitions
    - prototypes (declarations)
    - This is the `header file`, the `.h file`
- Implementation:
    - Full definition for every provided function
    - The `.cc file`

**Recall from C**

- declaration: asserts existence
- definition: full details, allocates space ( for `var` and `functions`)

**Example**

```C++
//vec.h (interface file)

struct vec {
  int x,y;
}; // definition of the type vec

vec operator+(const vec &v1, const vec &v2);
```

```C++
//main.cc (client)
\#include "vec.h"

int main() {
  vec v{1,2}
  v = v + v;
}
```

```C++
//vec.cc (implementation file)

\#include "vec.h"
vec operator+(const vec &v1, const vec &v2) {
  vec v {v1.x + v2.x, v1.y + v2.y};
  return v;
}

// Recall: An entity can be declared as many times as you want,
// but defined only once.
```

### Compiling Separately

**NEVER COMPILE** `**.h files**`

To do so, we use;

```Bash
g++ -c vec.cc  # -c means compile only, do not *link*, so above, only comiple vec.cc
g++ -c main.cc # does not build executable, instead, produces an object file (.o file)

g++ vecc.o main.o -o main
# link the object files together to produce executable main
```

What happens if we change `vec.cc`, we only need to recompile `vec.cc` and relink

But, what if we change `vec.h`? Now, we need to recompile `vec.cc` and `main.cc` since both files include `vec.h`, and then we must relink.

### How can we keep track of dependencies, and perform minimal compilation?

Linux tool: `make`

To use `make`, we create a `makefile` (must be named that), that says which files depend on which other files, and how to build your program.

```Bash
main: main.o vec.o \#target main depends on vec.o and main.o
  g++ main.o vec.o -o main # how to build to get main

# the space in front of the g++ MUST be a tab character

main.o: main.cc vec.h
  g++ -std=c++14 -c main.cc

vec.o: vec.cc vec.h
  g++ -std=c++14 -c vec.cc
```

```Bash
main: main.o vec.o \#target main depends on vec.o and main.o
  g++ main.o vec.o -o main # how to build to get main

# the space in front of the g++ MUST be a tab character

main.o: main.cc vec.h
  g++ -std=c++14 -c main.cc

vec.o: vec.cc vec.h
  g++ -std=c++14 -c vec.cc

# now with the below, it gets rid of the compiled files, removes everything
.PHONY: clean

clean:
  rm \\*.o main

# basically, telling make to run this "remove" command for us
```

Then from `cmd-line`:

```Bash
make # builds whole project

# - now if we just change vec.cc

make # compiles vec.o and relinks
make target \#builds the requested target, wihtout target (as above) builds the first
```

`make` just checks time stamps, if a target is older than its dependencies, it must rebuild (can cascade.recurse)

**lets generalize with variables**

```Bash
CXX = g++  \#compiler name
CXXFLAGS = -std=c++14 -Wall # compiler flags
OBJECTS = main.o vec.o
EXEC = main

${EXEC}: ${OBJECTS}
  ${CXX} ${OBJECTS} -o ${EXEC}

main.o: main.cc vec.h
vec.o: vec.h vec.c

# can now omit recipes, make assumes that the recipe is ${CXX} ${CXXFLAGS} -c *filename*.cc
```

Great, but the biggest problem is knowing dependencies

We can get help from `g++` with the `-MMD`flag, which produces compilation dictionaries

now,:

```Bash
CXXFLAGS = -std=c++14 -Wall -MMD # compiler flags

# Using this in the above block of code:

g++ -MMD -c vec.cc \#produces vec.o and vec.d
cat vec.d
vec.o: vec.cc vec.h
```

```Bash
# then, we can change:

CXX = g++  \#compiler name
CXXFLAGS = -std=c++14 -Wall -MMD
OBJECTS = main.o vec.o
EXEC = main
DEPENDS = ${OBJECTS:.o=.d} # so declares everything in main with .o and the end, and makes a .d file
${EXEC}: ${OBJECTS}
  ${CXX} ${OBJECTS} -o ${EXEC}

include ${DEPENDS}

\#WILL HAVE TO MAKE A "MAKE FILE" IN THE FUTURE FOR THIS COURSE
```

As project expands, only have to add `.o files`to the `OBJECTS` variable.