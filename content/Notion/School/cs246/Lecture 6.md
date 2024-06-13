```C++
\#include <iostream>
\#include <string>
\#include <sstream>
using namespace std;

int main () {
 int n;
 while (true) {
   cout << "Enter a number:" << endl;
   string s;
   cin >> s;
   istringstream ss{s}; // initialize a istringstream object to the string s
   if (ss >> n) break; // so if
   cout << "I said, ";
 }
 cout << "You entered " << n << endl;
}
// REVIEW THIS CODE
```

**Example** Read all integers until EOF and skip non-integers

```C++
int main () {
  string s;
  while (cin >> s) {
    istringstream ss{s};
    int n;
    if (ss >> n) {
      cout << n << endl;
    }
  }
}
```

---

**Command Line Arguments**

- Write a program that takes an integer and a string as command line arguments and prints the second integer argument a number of times equal to the first argument. *

e.x.

`./args 5 hello`  
output is expected to be:  

```C++
> hello
> hello
> hello
> hello
> hello
```

Here is the code:

```C++
int main(int argc, char** argv) {
  if (argc < 2) {
    cout << "usage: " << argv[0] << " int string" << endl;
  }
  string ${argv[1]};
  istringstream iss{s};
  int n;
  iss >> n;
  for (int i =0; i < n; i++) {
    cout << argv[2] << endl;
  }
}
```

---

**Here are some notes that I am making after the lecture**

- Above, we passed `argc` and `arv` to main, these are the command line arguments we can pass to main. Again, command-line arguments are given after the name of the program in command-line shell Operating Systems. Essentially, the command line arguments you used in bash.
- To pass command line arguments in C/C++, we need the `argc` and `argv`.
- `argc` (which can be though of as argument count) stores the number of command line arguments passed by the user, including the name of the program. So, its basically always the `number of arguments + 1`
- `argv` (which can be thought of as argument vector) is an array of character pointers listing all of the arguments.
- If `argc > 0` , then the array elements from `argv[0] to argv[argc-1]` will contain pointers to strings
- `argv[0]` is the name of the program, after that, every element in `argv` is command line argument

In fact, when we open a `C++` file in Xcode, the main you see looks like this:

```C++
\#include <iostream>

int main(int argc, const char * argv[]) {
    // insert code here...
    std::cout << "Hello, World!\\n";
    return 0;
}
```

---

**Good practice, implement cat**

**Default function parameters**

```C++
void printSuiteFile(string name="suite.txt") { // default value,
  // if no parameter is passed to the fnction, suite.txt is used as name
  ifstream file{name};
  string s;
  while (file >> s) {
    cout << s << endl;
  }
// rest of the function goes here, for printing the contents of a file
}
  printSuiteFile(); // pritns from suite.txt
  printSuiteFile(suite2.txt); // prints from suite2.txt
```

NOTE: Default parameters must be last  
e.g.  

```C++
int foo(int a = 0, int b, int c = 10) { // not allowed
 return (a+b)%c;
}

// foo(1,2) is 1=a, 2=b? or is 1=b, 2=c?
```

**Overloading**

- Same function name -> multiple possible data parameters - used in classes

e.g.

```C++
int neg(int a) {return -a;}
bool neg(bool b) {return !b;}
```

- Can have functions with the same name, so long as they differ in number or type of parameters
    - Differing in return type is **NOT** enough
    - *Can overload operators as well **

e.g.

```C++
// we can add strings and numbrs
string allan = "allan";
string brian = "brian";

string new = allan+brian;

int allan = 1;
int brian = 2;

int new_name = allan + brian;

// overload operator
```

**We've already seen overloading**

- Another example is `std::cin` and `std::cout`, can read in or output strings, integers... many data types
- This is **overloading**

---

**Structures**

```C++
struct Node {
  int data;
  Node *next; // compiler wont let you do this without pointer
}
```

**Constants**

```C++
const int MaxGrade = 100; // must be initializes
```

- Declare as many things as you can as `const`, it helps you catch errors

```C++
Node n1 = {5, nullptr};
// when want to set to nullpointer, DO NOTE use NULL or 0 anymore
const Node n2 = n1;
// const copy of n1
```

---

- *Parameter Passing **
- References (**Very important**)

```C++
int y = 10;
int &z = y

// z is an l-value reference to an int
// z is bound to y, and behaves exactly how y bahaves, so can just change it, and y will change as well
```

- we say that z is an `alias` for y
- It is important to recognize that references are **NOT** `pointers`
- They do behave like `const` pointers with automatic dereferencing
- So, in the above, `z++` will increment `y`
- if i add:

```C++
int y = 10;
int &z = y

int p=0;
z = p; // this does not mean z points to p, means y now equals 0.
++z; // now y and z = 1
```

**NOTE:** You can never change what z is a reference for  
  
**Also**:

```C++
cout <<< &y << endl; // prints out the address of y
cout << &z << endl; // prints out the address of y
```

**THINGS YOU CANNOT DO WITH AN L-VALUE REFERENCE**

- Cannot leave references uninitialized
    - cannot just say `int &bruh`
- Must be initialized with something that has an address
    - i.e. an l-value
    - so `int &lol = 3` is **not allowed **
    - Cannot be referencing a **r-value **
- create a pointer to a reference (it might not even have an address, so how can we have a pointer to it)
- however, we **CAN** have a reference to a pointer
- cannot create a `reference` to a `reference`
- cannot create an array of references
    - e.g. `int & r[3] = {n,n,n}`

**So, what CAN we do with references?**

```C++
void inc(int &n) {
  n++;
}

// now, n is a reference to the int passed in as an argument, so it
// behaves exactly like the argument, incrementig n increments the argument

int main() {
  int x = 5;
  inc (x);
  cout << x << endl;
  // so now, x will output as 6;
}
```

- So, why does `cin>>x` work?
    - because `>>` takes x by reference, i.e
    - `std::istream &operator>>(std::istream &n, in &x)`

```C++
int f(int n) {...} // a copy consturctor
// if the arg is big, could be expensive
```

Prefer pass by `const` reference over pass by value, unless, the function needs to make a copy anyways, then pass by value

**END OF LECTURE 6**