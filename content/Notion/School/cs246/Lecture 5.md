- EX/ Read all integers from stdin and echo them one per line
- Stop on bad input or EOF

```C++
int main() {
  int i;
  while (true) {
    cin >> i;
    if (cin.fail())
      break;
    cout << i << endl;
    }
}
```

Note, `>>` is `C`'s right bit-shift operator,

```C++
a >> b
```

- Shifts `a`'s bits to the right by b positions
- But, when the left hand side is an input stream, like `cin`, it is the input operator

The operator `>>`, when used for input, has `cin` (type istream) as its first operand and the data to be read in as its second (which could be of several different types).

- **But, what is the output (return value) of the operator? ***
- The return value is the stream itself (for example, `cin`)
- This is why we can write:

```C++
cin >> x >> y >> z;
```

`cin` reads into x, populates it, then this operator returns `cin` back

So, the above code becomes

```C++
cin >>> y >> z

// Then, cin reads into y, returns back cin, etc.
```

`cin` or istreams in general, can be implicitly cast to boolean values. `cin` is `true` if its `fail bit` is not set and `false` otherwise.

Something thats not that important now, but is nice to know, is that `cin` is an object in the `istream`class.

```C++
int main() {
 int i;
 while (true) {
   if (!(cin >> i))
     break;
   cout << i << endl;
 }
}
```

An even better way of re-writing the above:

```C++
int main() {
int i;
while (cin >> i) {
  cout << i << endl;
  }
}
```

**Example**: Read all integers from `stdin` until `EOF`, skip over bad input.

```C++
int main() {
  int i;
  while (true) {
    if (!(cin >> i)) {
      if (cin.eof()) {
        break;
      }
       cin.clear(); // clears the fail bit
       cin.ignore(); // removes the immediate next character from the stream
     } else {
      cout << i << endl;
    }
  }
}
```

- *What if I want to read in white space? **

we can use the header `\#include <iomanip>`. This header provides several io manipulators which can help us (listed below).

IO manipulator mutates the stream:

```C++
cout << std:hex // tells to print in hex, will continue to do so for all prints after this, if want to change, need to call std:dec
cout << std:dec // prints in decimal

cin >> std:noskipws // dont skip white spaces
cin >> std:skipws // skips white space
```

---

**STRINGS**

In `C`arrays, (`char * or char []`), terminated by the `null`character `\\0`

- Must explicitly manage memory - allocate more memory as string grows
- Easy to actually overwrite null terminator and corrupt memory
- Thankfully, we have built in type `string` in `C++`.
- we need include header:

```C++
\#include <string>

string s = "hello"
```

- `std::string`
- grows as needed (no need to manage memory)
- safer to manipulate
- In the above code snippet, even in C++, the literal `"hello"` is still a c-style string
    - i.e. a null-terminated array of characters
    - The variable s is a string whose value has been initialized using that c-string

**Perks of C++ Strings**

- equality: `s1 == s2`
- inequality: `s1 != s2`
- comparison: `s1 <= s2` and `s1 >= s2`
- length: `s1.length()` length of a string
- fetch individual chars: `s1[0]` etc...
- concatenation: `s3 = s1+s2, s3+=s4`

```C++
int main() {
  string s;
  cin >> s; // reads a string skips leading whiet space stops reading at next ws.
  cout << s;
}
```

**One way to read ws:**`getline(cin, s)` reads to the newline character into s

The stream abstraction applies to other data sources

- e.g. Files: read from a file instead of `stdin`:
    - In `C++`, we use `std::ifstream` to read from a file and `std::ofstream` to write to a file

---

**File access in C**

```C
\#include <stdio.h>

int main() {
  char s[256];
  FILE * = fopen("suite.txt", "r");
  while (true) {
    fscanf(file, "%256", s);
    if (feof(file)) {
      break;
    }
    printf(%s\\n, s);
    fclose(file);
  }
}
```

**File access in C++**

```C++
\#include <stdio.h>
\#include <fstream>

using namespace std;

int main() {
  ifstream file {"suite.txt"}; // declaring and initializing an ifstream to open the file
  string s;
  while (file >> s) {
    cout << s << endl;
  }
}

// as soon as ant ftream variable goes out of scope, the file is closed
```

Anything you can do with `cin` and `cout`, you can do with an `ifstream` and `ofstream`, respectively.

**Example**: The stream abstraction also applies to strings!

```C++
\#include <sstream>
```

Types `std::istring` stream for reading from a `string`, `std::ostring` stream for writing to a `string`.

e.g. convert a string to a number

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

What is the `stringstream`? Just like how `cin` and `cout`are read in from the `iostream`, `stringstream` makes a string just like a stream. That this, we can sort of imagine that string as an "input" and read from it. This could be useful when we want  
to extract strings within a string or extract integers from a string  

Basic methods are:

- `clear()`To clear the stream.
- `str()`To get and set string object whose content is present in the stream.
- operator `<<` - Add a string to the `stringstream` object.
- operator `>>` - Read something from the `stringstream` object.

```C++
/* Extract all integers from string */
\#include <iostream>
\#include <sstream>
using namespace std;

void extractIntegerWords(string str)
{
    stringstream ss;

    /* Storing the whole string into string stream */
    ss << str;

    /* Running loop till the end of the stream */
    string temp;
    int found;
    while (!ss.eof()) {

        /* extracting word by word from stream */
        ss >> temp;

        /* Checking the given word is integer or not */
        if (stringstream(temp) >> found)
            cout << found << " ";

        /* To save from space at the end of string */
        temp = "";
    }
}

// Driver code
int main()
{
    string str = "1: 2 3 4 prakhar";
    extractIntegerWords(str);
    return 0;
}
```

**END OF LECTURE 5**