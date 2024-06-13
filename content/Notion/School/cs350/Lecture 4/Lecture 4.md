### The POSIX File Interface Overview

Recall that the `stdio` library in **C** provides functions for working with files such as ==`fopen`==, ==`fscanf`==, ==`fprintf`==, ==`fseek`== and ==`fclose`==.

  

Most popular OS follow (or mostly follow) a standard that was originally meant for different versions of Unix (called POSIX — Portable Operating System Interface), they are built ontop of certain POSIX functions:

- `open` — returns a **file descriptor:** A file descriptor is **an unsigned integer used by a process to identify an open file**.
    - Other file operations (e.g., `read`, `write`, `lseek` and `close`) for that process require this file descriptor as an argument in order to identify that file from all the others.
    - Each open file (e.g. valid file descriptor) has an **associated file position - starts as byte 0**
- `close` — **invalidates the file descriptor**
    - The kernel tracks which file descriptors are currently valid for each process
- `read` — copies data from the file into the process’s **address space**
    - This copying is necessary because each process in a modern operating system operates within its own protected memory address space.
- `write` — copies data from process’s **address space** into a file
- `lseek` — enables non-sequential reading and writing.
    - `lseek` is used to reposition the file byte offset within the same file associated with a descriptor
    - It allows you to move the read/write pointer within the currently open file so that you can read or write data from a specific position within that file.
    - For example, if you have a file open and you want to read or write data starting from a particular byte offset within that file, you can use `**lseek**` to set the file offset to that specific position within the same file.

  

**POSIX** also provides some standard file descriptors that for files that are open by default for

each process. These include

- 0 for `stdin`,
- 1 for `stdout`,
- 2 for `stderr`

Normally these are all directed towards the console but can be redirected to read from or

write to files.

  

What if I want to redirect input from command line to a file? We can use `dup2` in this case:

```C
int dup2 (int oldfd, int newfd);
```

- **Makes** `newfd` an exact **copy** of `oldfd`
    - Same file descriptor and same byte offset (position in the file)
- **Closes** `newfd`, if it was already a valid file descriptor — need to close, and re-assigning it now, don’t want it to be re-assigned and the old file never closed
- If error occurs, this will return -1
- The two file descriptors will share same offset (`lseek` on one will affect both)

  

Consider the following code snippet:`

```C
fd = open(infile, O_RDONLY);
dup2(fd, 0);
```

Here is a line by line analysis of the above:

1. **`fd = open(infile, O_RDONLY);`**
    1. This line of code is attempting to open a file specified by the `**infile**` variable in read-only mode (O_RDONLY). It assumes that `**infile**` contains the path to the file that needs to be opened.
    2. The `**open**` function is typically used to open files in C. It returns a file descriptor (`**fd**`) that represents the opened file or an error code if the file couldn't be opened.
2. `**dup2(fd, 0);**`
    1. This line of code is using the `**dup2**` function to duplicate the file descriptor `**fd**` and associate it with file descriptor `**0**`. In Unix-like systems, file descriptor `**0**` represents the standard input (stdin).
    2. So, what this line effectively does is to make the file referred to by `**fd**` (which was opened in read-only mode in the previous line) the new standard input for the current process. Any subsequent reads from standard input (e.g., using functions like `**scanf**` or `**fgets**`) will read from the file specified by `**infile**`.

  

We can add more to our old `minish.c` which we saw last lecture. In particular, we want to add support for commands that redirect. An enhanced version our mini shell called `redirsh.c` will use `dup2` to help deal with  
commands that redirect input, output or error message. I.e. for a command such as:  

```C
    command < input > output 2> errlog
```

The function that parses the command line, `parse_input`, will now recognize that

- if `< input` is present then the file input should become stdin,
- if `> output` is present then the file output should become stdout,
- if `>2 errlog` is present then the file errlog should become stderr.

and call them `infile`, `outfile` and `errfile`, respectively.

  

Now, consider the following code:

```C
void doexec(void) {
	int fd;
	if (infile) { /* non-NULL for "command < input" */
		if ((fd = open(infile, O_RDONLY)) < 0) {
     perror(infile);
     exit(1);
		}
		if (fd != 0) { 
			dup2(fd, 0);
			close(fd);
		}
	}

	// ... do same for outfile→fd 1, errfile→fd 2 ..., so for outfile and errfile 
	execvp (av[0], av);
	perror (av[0]);
	exit (1);
}
```

Now, let’s analyze the code above:

- Before calling `doexec`, `infile` points to `NULL`. The function `parse_input` will look for a  
    substring of the form  
    `< input` and if it exists set `infile` to point to it.
- **Line 3:** If `infile` is not `NULL`, i.e. the input was redirected
- **Lines 4–7:** then try to open `infile` and report an error if it fails (i.e. a negative `fd`).
- **Lines 8–11:** If `fd` is not stdin then set stdin to correspond to this file and close the  
    original file  
    
- **Line 14:** repeat check for redirected output and error messages
- **Lines 15–17:** this is what the old version of `doexec` did in `minish.c`

  

### Pipes

Recall from that in the shell script, pipes allow us to connect the stdout of one command to the stdin of another command, providing seamless data flow between them. We will look at how POSIX defines these. Consider the following code:

```C
int pipe (int fds[2]);
```

- Passed two file descriptors in `fds[0]` and `fds[1]`
- The output of `fds[1]` becomes the input to `fds[0]`, i.e. `fds[1] | fds[0]`
- When last copy of `fds[1]` is closed, `fds[0]` will return EOF (last copy being closed indicates no more data will be written into the pipe, and so, `fds[0]` will behave as if it had reached the end of the file).
- Returns 0 on success, -1 on error

  

Operations on pipes

- `read`, `write` and `close` – work the same as with files.
- When `fds[1]` closed, `read(fds[0])` returns 0 bytes,
- When `fds[0]` closed, `write(fds[1]`):
    - Kills process with `SIGPIPE`
    - Or if signal ignored, fails with `EPIPE`

  

We can see an example of this is implemented in `pipesh.c`:

```C
void doexec (void) { 
	while (outcmd) {
		int pipefds[2]; 
		pipe(pipefds); 
		switch (fork()) {
			case -1: // error
				perror("fork"); exit(1);
			case 0: // child writes to the pipe
				dup2(pipefds[1], 1); //stdout is pipe input
														 // anything written to stdout in the child process 
														 //will be sent to the write end of the pipe.
				close(pipefds[0]); 
				close(pipefds[1]); 
				outcmd = NULL;
				break;
			default: // parent reads from the pipe 
				dup2(pipefds[0], 0); // stdin is pipe output 
				close(pipefds[0]); 
				close(pipefds[1]); 
				parse_input(&av, &outcmd, outcmd);
				break; 
		}
}
```

Now, this code basically, splits off using `fork()` into child and parent, where each one either does the write or read. Here is a more in-depth analysis of the above code:

- **Line 2–18:** Loop through the different pairs of commands, e.g. `cmd1 | cmd2 | cmd3`
- **Line 2:** `outcmd` represents the sequence of commands the pipe(s) connect. It will loop until there are no further commands to output to. The child will set `outcmd` to NULL after sending its output the pipe. The parent will not set it to `NULL` but will continue in the loop to send output to `cmd3`.
- **Line 3:** Execute the system call pipe to set up the two ends of a pipe, say `cmd1 | cmd2`
- **Lines 4–6:** Call fork and report the error and exit if it fails
- **Lines 7-11:** The child will handle cmd1 and sets the pipe’s input to its `output`, `stdout`
- **Lines 12–14:** The parent will handle `cmd2` by setting the pipe’s output to its `input`, `stdin`
- **Lines 15:** It will then continue on if there are more commands in the sequence of pipe statements

  

So, why do we fork? It is occasionally useful to fork one process. The main reason we use it here is for _simplicity of interface:_

- There are many things you might want to do to child:
    - Manipulate file descriptors, environment, resource limits, etc.
- Yet fork requires no arguments at all

  

### Implementing Processes

The OS **maintains data about each process —** typically called the **Process Control Block** (PCB) but also called `proc` in Unix and a `task_struct` in Linux.

- It tracks the **state** of the process
- Includes information necessary to run
    - Registers, virtual memory mappings, etc.
    - Open files (including memory mapped files)
- Various other data about the process
    - Credentials (user/group ID), signal mask (which signals are being blocked), controlling terminal, priority, accounting statistics, whether being debugged, etc.

  

Consider the visuals below:

![[Screenshot_2023-09-26_at_10.12.47_AM.png]]

![[Screenshot_2023-09-26_at_10.13.01_AM.png]]

Here is a rundown of the different states:

- `New` — This state represents the initial state of a process when it is created or spawned. At this stage, the process is being set up, and resources are being allocated to it. It transitions to the "ready" state when it's ready to execute but hasn't been scheduled to run yet.
- `Ready` — Processes in this state are ready to execute but are waiting for their turn to run on the CPU. They are waiting in a queue, and the operating system scheduler will select one of them to transition to the "running" state when the CPU becomes available.
- `Running` — A process is in the "running" state when it is actively executing its instructions on the CPU. This is the state where the process is actually doing its work. It may transition to other states when it voluntarily yields the CPU (e.g., for I/O) or when it is preempted by the operating system scheduler.
- `Waiting` — When a process is waiting for some event or condition to occur before it can proceed with its execution, it is in the "waiting" state. For example, a process waiting for a disk I/O operation to complete or waiting for user input would be in this state. Once the required event occurs, the process may move back to the "ready" state if it's ready to execute or directly to the "running" state if the CPU is available.
- `Terminated` — When a process has completed its execution or has encountered a fatal error, it transitions to the "terminated" state. At this point, the process has finished its work and may be removed from the process table and its resources freed.

  

Here we ask, how does the kernel determine which processes run in what order? Obviously, if there are no ready programs, we won’t run anything, or if there is only one process ready, we will only run that one process. But if we have > 1 ready, we need to make a scheduling decision.

- **Scheduling** is the method used to decide which process to run.
- **Idea:** Divide into runnable and waiting processes and use FIFO/Round-Robin on the list of runnable processes.
    - Add back to list and dispatch them from the front, like **queue**
        
        ![[Screenshot_2023-09-26_at_10.35.31_AM.png]]
        

  

I mentioned **preempting** above, but just what is preempting — refers to the action of temporarily interrupting the execution of one task or process to allow another task or process to run. The primary purpose of preemptive scheduling is to ensure fairness, responsiveness, and efficient resource utilization in a multitasking environment.

  

**⇒ When to preempt?**

- Can preempt a process when kernel gets control:
    - System call, page fault (data in swap space), illegal instruction, etc.
    - May put current process to sleep, e.g., when reading from a disk.
    - May make other process runnable, e.g., fork, write to pipe.
- Have periodic timer interrupts and if a running process used up quantum, schedule another.
- Device interrupt
    - Disk request completed, or packet arrived on network.
    - Previously waiting process becomes runnable.
    - Schedule if higher priority than current running process.
- Changing the running process is called a **context switch**.  
      
    

**⇒ Context Switch**

Context switching is a fundamental operation in operating systems that involves saving the current state of a running process (or thread) so that the operating system can switch to executing a different process or thread. It allows the operating system to manage multiple processes or threads concurrently on a single CPU (or core).

![[Screenshot_2023-09-26_at_10.50.54_AM.png]]

![[Screenshot_2023-09-26_at_10.51.01_AM.png]]

Some notes on this:

- **Current State —** When a process is executing on a CPU, it has its own set of registers, program counter, stack pointer, and other processor-specific state information. This state represents the exact point at which the process is currently executing.
- **Need for Switching —** Context switching is necessary when the operating system scheduler decides to pause the execution of the currently running process and start executing another process. This decision can be based on various factors, including time-sharing policies, priority levels, or the need to give each process a fair share of CPU time.
- **Saving State —** To perform a context switch, the operating system first saves the complete state of the currently running process. This involves saving the values of registers, the program counter, and other relevant CPU state information into PCB (Process Control Block)
- **Loading State —** After saving the state of the current process, the operating system retrieves the state of the process to be scheduled next from its corresponding PCB
- **Switching Execution**: The operating system then updates the CPU's state with the saved state of the new process. This includes loading the program counter, registers, and stack pointer with the values from the new process's PCB or TCB.
- **Resuming Execution**: Once the state of the new process has been loaded, the CPU resumes execution from the point where the new process left off. It continues running this process until the next context switch is required.
- **Overhead**: Context switching introduces some overhead because it involves saving and loading CPU states and updating data structures. Minimizing this overhead is important for efficient multitasking.