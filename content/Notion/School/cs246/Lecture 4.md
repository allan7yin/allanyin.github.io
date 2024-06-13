How many times does a word `$1 (first cmd line arg)` occur in a file `$2 (2nd cmd line argument)`

```Bash
#!/bin/bash
x=0
for word in $(cat $2); do
  if [ $word = $1 ]; then
       x=$((x+1))
  fi
done

echo $x
```

For example, suppose that I have a `file.txt` that contains section of the Apple Wikipedia page. By entering the following into the command line:

```Bash
./glow.txt "Apple" file.txt
```

The output will be:

```Bash
14
```

---

**EX/** Payday is on the last Friday of the month. Compute this month's payday

```Bash
#!/bin/bash

report() { # indie a fn $1 on denote the "fn's" parameters

        if [ $1 -eq 31]; then
            echo "This month: the 31st"
        else
            echo "This month ${1}th"
        fi
 }

 awk '{print $6}' # this prints the 6ht column, from assignment 1
```

---

```Bash
report() { # indie a fn $1 on denote the "fn's" parameters

        if [ $1 -eq 31]; then
            echo "This month: the 31st"
        else
            echo "This month ${1}th"
        fi
 }

report $| cal $1 $2 |  awk '{print $6}' | egrep "[0-9]" | tail -1
# cal is cmd for a calender
```

---

- Note that `$1` outside of report it was the scripts fist cmd line arg, and in report, it was the fn's cmd line arg.
- `$0` doesn't change as it is the currently running program, and the fn is not a program

---

**WE ARE NOW DONE WITH BASH**

---

- Testing is an essential part of program development
    - Ongoing - not just test at the end
    - Testing begins before you start coding
    - Test suites test expected behaviour
    - Continue to test while you code - includes writing new tests
- Testing is NOT debugging - cannot debug without testing
    - Testing cannot guarantee correctness - can only prove wrongeness
    - Testing is not easy
        - There is no general formula
        - psychological barrier - don't want to find out program is wrong - more work unlucky

Ideally, developer and tester should be different. Ofc, not for this course

- Types of testing:
    - Human Testing, look over code, find flaws
    - code inspection, walk-through
- Machine testing:
    - run program on selected input, check against spec
    - can't check everything, check test cases carefully
    - Black/white/gray box testing - no/full/some knowledge of the program
        - Black box testing, no knowledge of implementation - making test cases
        - White box testing, after knowing what program does, making tests, test every possible path of control flow in the program
        - Test with various classes of input, ranges, only whats valid input, boundaries, edge cases
- `* DON'T TEST INVALID TEST **`

Unless a behaviour has been prescribed, if the input is invalid, and there's no specified behaviour, then there is no such thing as correct output and any such test case is meaningless

- Regression Testing:
    - Make sure new changes toe the program don't break old functionality
    - Typically achieved though test system testing script
    - Always add never subtract test cases

---

**C++**

---

- return statement returns the status code to the shell (in main)
- if not return 0, and program reaches end of main, default returns 0

**Compiling C++ programs (student cs environment)**

```Bash
g++ -std=c++14 program.cc -o program
```

The `-o` program indicates the name (program here) of the resulting executable program. If not specified, the compiler uses a.out

_**Most C programs work in c++**_

---

**Input/Output**

- There are 3 io streams
    - `cout` and `cerr` for printing to `stdout`/`stderr` respectively
    - `cin` for reading in from `stdin`

to add two numbers

```C++
    int x,y;
    std::cin >> x >> y;
    std::cout << x+y << std::endl;
```

- what if we `cin` something not an integer?
- What if the input DOES NOT contain an integer next or the integer is too small/large?
- What if we run out of input before we get two `int`s?
- *When the above happens: **
- Read fails, value of `var` is unchanged if read fails (max/min int if the large/shell)
- If the read fails, `cin.Fail()` will be true (fail is apart of `std` class)
- If the read fails because of `EOF`, the both `cin.fail()` and `cin.eof()` will be true -- but neither are true until read fails!

**END OF LECTURE 4**