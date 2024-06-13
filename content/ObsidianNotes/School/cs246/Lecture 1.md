- ssh stands for secure shell
- *Linux shell **
    - An interface to the operating system, gets the **OS** to "do things"
    - run programs, manage files
- **Two types of shell:**
    - graphical shell: point and click, click to run programs
    - textual shell/command line: type commands and more versatile, for me, the **terminal**

**Basic Bash Syntax**

- cd:current dir
- cd .. (this notation moves back one directory)
- cd ~ (goes back to home directory)
- cp file dir -> copies file into directory named dir
- mv old.txt new.txt , this moves or renames files
- ls - lists all non-hidden files in the current directory (cwd)
- pwd - prints the current working directory
- cat/user/share/dict/words // if no file is found, reads from standard io
- man is manual for a command
- ls -d .* lists all hidden files
- ls -a lists all files, hidden and non-hidden
- ctrl c kills program
- ctrl d tells cat I am done typing
- clear clears the ui
- wc gives count of lines words characters file name

In bash, > is used for output redirection

```Bash
cat > glow.txt
```

when no command line argument is passed to cat, it automatically defaults to the standard input, the keyboard. The keyboard input is then redirected into the file glow, which if we cat, we could see its content.

```Bash
This is being redirected from standard ouput to the file glow.txt
```

- cat is the program and "/user/share/dict/words" is first command line argument
- cat is a command that prints the contents of the files printed as the command-line arguments
- cat without CLA (command lie arguments), reads from the user and prints to the screen what it reads in
- To stop, hit ctrl d

  

**What are command-line arguments?**

- command-line arguments are extra texts written after the program you want to execute before you hit enter and send that command to the OS
- above, words is a file in the dict directory
- dict is then a directory in the share directory which is then in the user directory

```Bash
"/user/share/dict/words" is path/file path -> path to a file
```

Any path that starts with a / is an absolute path. The first slash denotes the root directory, top most directory of the system, everything in the root directory - one big tree

In linux, every file must end with a new line character. Marmoset will check for this.

We can also redirect input with the < sign

```Bash
(1) cat input_file.txt vs (2) cat < input_file.txt
```

- (1) has a cla, string "file.txt" is passed as cmd line argument to the cat program. Cat then opens input_file.txt and prints the contents of the file
- (2) has no cla, so cat expects input from user, but os passes it input_file.txt since we gave it to it os opens the file and passes the content to cat to output

Another example:

```Bash
 (1) wc file.txt vs (2) wc < file.txt
```

- (1) outputs 2 3 4 file.txt
- (2) ouptuts 2 3 4

**Combining IO redirection**

We can combine input and output redirection

```Bash
cat < filt.txt > second_file.txt
```

The above effectively copies the contents of file into second_file

```Bash
cat >> file.txt
```

The above appends input onto the file's end.

- Since we can pass multiple cla to cat, could cat file.txt second_file.txt, prints/reads from left to right
- so, we use cat \*.txt to print, cat asks **OS** to replace * with any file in cd with txt
- This use of \* is called **"globbing pattern"**

```Bash
pwd
\#that prints the current directory
ls -l
\#lists out elements of current directory, including permissions
cat *.txt
\#this prints out all files ending with a .txt
```

- In the context of globbing patterns, * is a wildcard character that means "match any sequence of character". When the os encounters a globbing pattern, it find all files in the cwd that fits the pattern
- cat -n: -n is flag that asks programs to change their rules, for cat, -n prints out line numbers and tabs in
- man function provides the manual

```Bash
man egrep
man cat
```

- every process (aka running program) is attached to three strings
    - std in -> program -> stdout or progam -> stderr
    - stdin - standard input, by default connected to the keyboard, can be redirected with <
    - stdout - standard output, connected by default to the screen, can be redirected with >
    - stderr - standard error, connected by default to the screen, can be redirected with 2>

stdout is buffered, so lets say i want to print a file, we put all those characters into a large array, a buffer, to make processing more efficient. This buffer is flushed every new line. stderr is not buffered

// TO SUBMIT A FILE TO MARMOSET

[https://student.cs.uwaterloo.ca/~cs246/current/marm_sub/index.html](https://student.cs.uwaterloo.ca/~cs246/current/marm_sub/index.html)

After ssh into school server, type:

```Bash
source /u/cs246/setup
marmoset_submit [course] [assignment] [solution file]
```

**END OF LECTURE 1**