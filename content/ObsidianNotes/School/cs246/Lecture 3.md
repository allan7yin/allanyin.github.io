File permissions:

- ls -al lists long form of listing alongside of hidden files, of all files in current directory
- ls -l shows: `permissions owner group size (bytes) datelastmodified filename`

For permissions: it is a 10 digit string

```Bash
 -rw-r----- # first first char, then it is user group other
```

```Bash
It goes read, write, execute
```

The first character in permissions is a "type" can be - (file) and a d means (directory)

Permissions are three groups of 3 bits and when all set, are:

```Bash
(-|d)rwxrwxrwx
```

1. user bit applies to files owner
2. group bit applies to files group
3. other bit applies to everyone else

```Bash
r: read
w: write
x: execute
```

---

**For DIRECTORIES:**

- **r:** means we can ls the directory, use globbing, and tab completion
- **w:** we can add or rm files from the directory
- **x:** means the directory can be navigated (can cd into it). If a directory's executable bit is not set, regardless of the other read or write bits, access to the that directory, nor any file within it, are set

---

- * for FILES:**
- **r:** can be read
- **w:** contents can be modified
- **x:** file's contents can be executed as a program

---

How can we change permissions? We have a cmd for that:

```Bash
chmod Mode file
```

The mode consists of 3 parts:

- User types:
    - u = user
    - g = group
    - o = other
    - a = all
- operator:
    - + add permissions
    - - subtract permissions
    - = set permissions exactly (the equal signs not in quotes are simply relay the definitions of the symbols, will lose other permissions that prev. existed
- permissions
    - r = read
    - w = write
    - x -executable

```Bash
\#e.g. say we ls -l and get this:

ls -l

-rwx------ 1 a7yin cs246  77 May  9 13:15 testing.txt
```

```Bash
\#running:
chmod u=rwx,go+rw testing.txt
```

This will give group and other read and write permissions. The resulting permissions looks like:

```Bash
 -rwxrw-rw- 1 a7yin cs246  77 May  9 13:15 testing.txt
```

---

**Shell Scripts**

Files containing a sequence of shell commands executed as a program

```Bash
Example: Print the date, current user, and current directory
```

```Bash
 #!/bin/bash -> shebang line
```

This line tells OS `#!/bin/bash` is the location of the program you should use to interpret this file.

Give the file executable permissions:

- `cd..` means go back one directory
- `cd../` means go back to the parent directory of the cwd

---

**Bash can have variables**

```Bash
x=1 #(no spaces here)
echo $x #(dollar means fetch the value of)
```

So `$` with no brackets is fetch value, with bracket is subshell

**NOTES:**

- Use $ when fetching the value of variable
- No $ when setting a variable
- Good practice:
- `${x}` also means fetch the value of x
- This is good practice because if I want to print "1st" `echo $xst` wont work, need echo `${x}st`

**Ofc** putting the above substitution inside of '' will not change it

---

**LAST NOTE:**

All bash variables are strings, e.g. x contains the string "1"

```Bash
$expansion
```

Occurs in the double quotes, but not in single quotes

- *Special was: **

```Bash
$1 $2,..,etc - cmd line arguments
```

Example: check whether a word is in the dictionary or not

In our file, we would write:

```Bash
 #!/bin/bash
 egrep "^${1}$" user/share/dict/words
```

So, we `egrep` any line in the dictionary (every line is just the words found in a dictionary) that starts and ends with the first command line argument.

e.g.

```Bash
./isitaword hello
```

This prints nothing if word not found, otherwise, prints the word

**Example:** A good password should not be in the dictionary

```Bash
 egrep "^${1}$" user/share/dict/words > /dev/null
 #(output where we dont care about)
# redirecting stout to /dev/null supresses stdout.
# Every program returns a status code to the OS when finished
# $? - the satus of the most recently executed command

# how to interpet this return alue? we can use ifs

 if [ $? -eq 0]; then #(THE SPACES ARE VERY IMPORTANT)
    echo Bad password
 else
    echo Maybe not a terrible password
 fi
```

- * WE NEED TO END IF STATEMENTS WITH FI**

As seen above, a return value of 0 is success and anything other than a 0, is a fail. So in the above code snippet, as the passed password command line argument is not in the dictionary, the return value is not 0.

```Bash
 -eq is for arithematic equality
```

- *Example: ** Verify correct # of args, print how to use the program otherwise

```Bash
#!/bin/bash

egrep "^{1}$" /user/share/dict/words ? /dev/null
```

(1) usage: write a function. good idea to make usage of a fn, in case you need # to print it multiple place

`$0` is current running program

---

```Bash
 #!/bin/bash
 usage() {
            echo "usage: ${0} password"
        }

 if [ $# -ne 1]; then
    usage
    exit 1
 fi
```

`$#` is a special variable in bash, that expands to the number of arguments (positional parameters) `e.g $1, $2 ...` passed to the script in question or the shell in case of argument directly passed to the shell

---

```Bash
if .... elif .... elif ....else
```

For all available comparisons and other conditions, see the reference sheet given on Learn.

**LOOPS:**

Print numnbers from 1 to $!

```Bash
 #!/bin/bash

 x=1
while [ $x -le $1]
do
  echo $x
  x=$((x + 1))       #$((...)) sytnax for arithematic
done

# Looping over a list

 for x in a b c; do echo $x; done

# (prints a b c)

# for loops

\#version 1
for x in a b c; do
  echo $x; done
# prints out a (new line) b (new line) c

# we need the ; when we write this function on one line, else, it
# could look like this.

\#version 2
for x in a b c; do
  echo $x
done
# personally, I like the second option better, more spaced out

for x in "a b c"; do echo $x; done
# prints out a b c (the "" make it an entire string)

for x in $(ls); do echo "File:$x"; done
# prints out File: and the name of every file in the current dir.
```

---

_The loop below will rename all .cpp files to .cc_

```Bash
for name in .cpp; do
   mv $name ${name%cpp}
   # ${name%cpp}  menas give me everything in name without the cpp
done
```

To run a file in bash, we need to give ourselves executable permissions to the file through `chmod`. Then, we need ./ in front of the executable file since we need the path to the file to run it.

**END OF LECTURE 3**