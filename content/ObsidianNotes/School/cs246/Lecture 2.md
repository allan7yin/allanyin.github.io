**Quick Note**

- stdout may be buffered - system can accumulate output before sending it out to os for printing
    - can save time as "context switching" to give the os control to d something (e.g. printing  
        or allocating memory for the program is expensive.  
        
    - allows for error messages to be sent to a separate location, regular output can maintain  
        its format  
        

What if we want to run one program with the input being the output of another program

program1 > filename

program2 < filename

**Pipes:**

Pipes allow us to use the output of one program as the input to another (io redirection uses only files, pipes uses programs)

e.g. How many words occur in the first 20 lines of Alice in Wonderland?

- `head -n` file, gives first n lines of file
- `wc` counts words, lines, characters (in that order). Can provide flags to make it count what we only want.
- `wc - w` couns only words

```Bash
In bash, the | is the piepline character
```

Below is an example of how we could use the pipe:

```Bash
egrep "cs246" course-outline.txt | head -10
```

So this pipe notation redirects the things on the left to the things on the right

```Bash
head -20 alice.txt | wc -w
```

Let's say we have a words and we want to delete duplicates  
So,  
`uniq` only looks at adjacent lines  
so, we must:  

```Bash
cat words.txt | sort | uniq
```

So, must sort the file first before removing duplicates.

Therefore, we can use the output of one program as the input to another. So, can we also use it as the arguments of another program?

  

**Yes, yes we can.**

  

We do so by using a sub-shell.

**Subshell**

A subshell is created with the `$(...)` in the parentheses goes whatever command you want to run, then the subshell gets replaced in the outers cmd with the text if printed

```Bash
echo - prints out its command line arguments, literally an "echo"

if you just type echo in the command line, echo just repeats
back to you what you type to it.
```

```Bash
echo "Today is $(date) and I am $(whoami)"
```

- `date` is cmd for the current date
- `whoami` is a cmd for whoever is logged into the shell

```Bash
a7yin@ubuntu2004-008:~/cs246/1225/a1$ echo Today is $(date) and I am $(whoami)
```

This outputs:

```Bash
Today is Sun 08 May 2022 01:00:24 PM EDT and I am a7yin
```

Here, the shell executes the cmds `date` and `whoami`and substitutes their output into the command line

`''` and`""`are both strings in bash

Difference is, use `''` to prints strings that have special characters in bash, `''` suppresses all bash substitutions

```Bash
echo 'Today is $(date) and I am $(whoami)'
```

Now, this outputs something different from earlier:

```C
Today is $(date) and I am $(whoami)
```

`echo` accepts a string, check `man echo` for more details  
for  
`echo`, a string is any non-white space character, so, if i have file.txt:

```Plain
 Hello
 I
 am
 Allan
```

```Bash
echo $(cat file.txt)
\#outputs: Hello I am Allan
```

**IF YOU WANT THE OUTPUT TO MAINTAIN ITS FORM, JUST CAT THE FILE**

**In general, NEVER DO ECHO(CAT...)**

```Plain
"" and '' suppress globbing (star) characters. e.g:
 echo \\ prints all non-hidden file names in current working directory
```

**PATTERN-MATCHING** in Text files

- `egrep` (extended global regular expression print)
- the cmd is: `egrep pattern file`
- This prints every line in the file that matches the pattern

e.g. print every line in index.html that contains the string "cs246"

```Bash
egrep "cs246" index.html #(dont need quotes)
```

This is case sensitive, so we can do:

```Bash
egrep "(cs|CS|Cs|cS)246" index.html # the | means or here
```

Alternatively, I could:

```Bash
egrep"(c|C)(s|S)246" index.html
```

The BEST way, however, is to:

```Bash
egrep "[cC][sS]246" index.html
```

The "patterns"that `egrep` understands are called regular expressions, and are NOT the same as globbing

```Bash
# [...] says match any one character in these brackets
# [^...] says match any character NOT in these brackets
```

- `[cC][sS] ?246` allows for an optional space before the 246, the ? indicates 0 or 1 occurrences of the immediately preceding pattern.
- The syntax indicates 0 or In general, never do echo(cat...) of the immediately preceding pattern
- Also, Matches any string (including an empty string)

**The syntax + indicates 1 or more of the immediately preceding patterns**

```Bash
a matches nothing, a,aa,aaa,aaaa,...
a+ matches a,aa,aaa
```

The syntax . matches any single character

```Bash
. matches any string including empty
.+ matches any non empty string
```

**EXAMPLES:**

In order to `egrep` a special character, need the `\` in front of it, still needs `''` wrapped around the entire string

```Bash
egrep "cs?246" file.txt
```

The `?` only applies to the s, c followed by 0 or 1 s, then 246

```Bash
egrep "cs.246" index.html
```

This fetches lines from index.html that contains cs followed by any sequence of characters followed by 246

The special patterns `^` and `$` (obv not in square brackets) match respectively the beginning and end of the line, thus, the pattern `^cs246$` matches any line that begin and end with _**EXACTLY**_ cs246 and nothing in between.

```Bash
		^cs246 matches any line that starts with cs246

Anything after that, we don't care

cs236$ matches any line that ends with cs246
```

1. Example: Fetch all lines of even length from a file

```Bash
egrep '^(..)$' file.txt
```

```Bash
/s is white character
/S is non-white character
```

1. Example: List files in the current working directory whose names contain exactly one a

```Bash
egrep "^[^a]a[^a]&" path
```

1. Example: Fetch all words in the global dictionary that are five letters long and begin with e

```Bash
egrep "^e....$" /user/share/dict/words
```

**END OF LECTURE 2**