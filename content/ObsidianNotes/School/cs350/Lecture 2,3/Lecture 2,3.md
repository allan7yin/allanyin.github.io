# Processes

So, what are processes? Here is a really easy example to think about. Consider the application Chrome on your laptop. Before you run it (click it), it is not a process, simply some executable file. But, the moment you launch it, and the application starts running, that is one process that is running on your computer’s OS — It is a **instance of a running program.** Again, here is a small diagram to help you understand:

![[Screenshot_2023-09-15_at_3.23.23_PM.png]]

Processes are one of the abstractions that are commonly part an operating system, which could also include threads, synchronization, address spaces and file systems. Processes are not as easy to recognize as you’d might think. An interesting way to visualize this is to through how the **Notepad** and **Word** applications work.

- Each time notepad (a simple text editor) is launched in Windows 10, it creates a new process.
- After MS Word is launched in Windows 10, the OS recognizes that Word is running and does not create any additional process each additional time it is launched. With this, Word maintains a constant (2 ~ 3) process, no matter how many new windows you open — it is smart enough to know you will never be typing in 2 places at once, and so, no need to create extra idle processes.

  

Modern OS run multiple processes simultaneously. Why?

- **Simplicity of programming:** a programmer does not have to account for the fact that  
    other processors may be running.  
    
- A hinted above, with the discussion on browsers, it is done for **robustness**. A program with bugs should not crash other processes or the OS.
- Higher **throughput**, i.e. better processor utilization (time being used vs. time waiting for  
    I/O etc).  
    
- Lower **latency**: time between launching a program and getting a response.

  

How else can we use multi-tasking? Multitasking can increase processor utilization by overlapping one processes computation time with another's time waiting for I/O. A good example of this is in the diagram below:

![[Screenshot_2023-09-16_at_4.54.07_PM.png]]

What does this mean? Consider when we are typing. The time in-between keystrokes, maybe like 10ms, the computer can get a lot done in that time. So, while the computer is waiting for input, it can concurrently run other processes.

- A process is **I/O bound** if the time it takes to complete a task is largely determined by the  
    time waiting for input or output.  
    - e.g. Word spends most of its time waiting for user input
- A process is **CPU bound** if the time it takes to complete a task is largely determined by  
    the speed of the processor.  
    - scientific computation
- If both A and B are completely CPU-bound, then the system cannot take advantage of  
    running one of them while the other is waiting for I/O.  
    

### **Concurrency, Parallelism and Preemptive Multitasking  
  
**

**Concurrency** means multiple programs or sequences of instructions running, or appearing  
to run, at the same time. It comes in two forms.  

1. **Parallelism** run at the same time because of multiple core processors or multiple  
    processors (My MacBook is multi-core)  
    
2. **Preemptive multitasking** appear to run at the same time but are actually rapidly  
    switching between them on the same hardware. Sometimes called timesharing.  
    

  

When I worked with threads in Java, the above was being done. The OS is responsible for deciding when to execute “true” multi-threading, and when to alternate between threads.

  

---

### A Process’s View of the Computer

Each process has own view of the computer with its own

- address space
- open files
- virtual CPU (through preemptive multitasking).

An address that each process sees is a virtual address. I.e. the same virtual address refers to different physical addresses in P1 and P2 (something we’ve seen in 251). There are 2 views of a process:

![[Screenshot_2023-09-19_at_11.41.20_AM.png]]

- The **user view** of processes.
    - Introduction to Linux/Unix system call interface
    - learn some process management function, e.g. how to create, kill, and communicate between processes
    - The running example is how to implement a shell
- The **kernel** view of processes
    - Implementing processes in the kernel

  

Let’s take a look at some commands related to **creating, deleting, and running processes**

**⇒ Creating Processes**

- `int fork (void);`
    - **==_Create a new process that is an exact copy of current one._==**
    - Returns process ID of new process in “parent”
    - Returns 0 in “child”
    - Gets called once and returns twice: once in parent and once in child
- `int waitpid (int pid, int *status, int options);`
    - ==_**Attempt to get exit status of child.**_==
    - pid: process ID to wait for, or -1 for any
    - status: will contain exit value, or signal
    - options: 0 means wait for the child to terminate (exit or crash) or `WNOHANG`` which  
        means do not wait for the child to terminate.  
        
    - Returns process ID or -1 on error

  

**⇒ Deleting Processes**

- `void exit (int status);`
    - ==_**Have the current process exit (like returning from main).**_==
    - status: shows up in an encoded format if `waitpid` is called.
    - By convention, status of 0 is success, non-zero error.
- `int kill (int pid, int sig);`
    - ==_**Sends signal sig to process pid.**_==
    - A **signal** is a software interrupt (i.e. interrupts the running of the program).
    - SIGTERM most common value, kills process by default
        
        (but application can catch it for “cleanup”)
        
    - SIGKILL stronger, kills process always.

  

**⇒ Running Programs**

- `int execve (char *prog, char **argv, char **envp);`
    - ==_**replaces the current running program (that called**_== `==_**execve**_==`==_**) with a new one prog.**_==
    - `prog`: full pathname of program to run
    - `argv`: argument vector that gets passed to main (It provides the command-line arguments to the new program being executed. The first element (`**argv[0]**`) typically contains the name of the program itself (conventionally).
    - `envp`: environment variables, e.g., PATH, HOME

  

When you call `**execve**`, the operating system attempts to load and execute the program specified by `**prog**`. If successful, it replaces the current process's memory and code with the new program, passing the provided command-line arguments (`**argv**`) and setting up the environment variables (`**envp**`) as the new program's environment.

If there's an error during execution, the function typically returns -1, and you can check the value of `**errno**` to determine the specific error that occurred.

  

`execve` is generally called through wrapper functions, such as:

- `int execvp (char *prog, char **argv);`
    - Searches PATH for prog, using current environment
- `int execlp (char *prog, char *arg, ...);`
    
    - list the arguments one at a time, finish with NULL
    
      
    

Let’s see some code that will help demonstrate this to us. Consider the following:

```C
pid_t pid; 
char **av;

void doexec() {
	execvp(av[0], av); // if execvp is successful, program av[0] will run instead
	perror(av[0]); // if unsuccessful, perror will run 
	exit(1);
}

// here is the main loop 
for (;;) {
	parse_input(&av, stdin);
	switch (pid = fork()) {
	case -1:
		perror("fork"); // if the fork is unsuccessful, perror will run 
		break;
	case 0
		doexec();    // if successful, child will run this 
	default:
		waitpid(pid, NULL, 0); // if successful, parent will run this 
		break;
	}
}
```

So, what is `fork()` doing in the above example? In the above, at the line `switch (pid = fork())`, a child process is created when this runs successfully. After this, we have 2 processes (imagine 2 versions of the code) running at the same place (the fork line). So, the parent will have the child PID as its `pid` value, and the child will have `0` as its `pid` value. Let’s break down the code above line by line:  
  

- **Line 1:** `av` is the argument vector. By convention, `av[0]`is the name of the program
- **Line 4:** `perror` prints the last error encountered during a call to a system or library function. If `execvp` is successful, this will not execute. The program specified by `av` will.
- **Lines 2–6:** attempts to execute the program specified by `av[0]` with the arguments
    
    specified in the rest of `av`.
    
- **Line 10:** `parse_input` converts the input from a single string (e.g. `ls -l docs`) into  
    an  
    `NULL` terminated array of pointers to strings ( e.g. `av[0]=“ls”`, `av[1]=“-l”`,  
      
    `av[2]=“docs”` and `av[3] = NULL`).
- **Line 11:** when `fork()` is called either
    - an error occurs, which is reported on lines 12–13 or
    - a child is created (say `pid=6`), which executes lines 14–15 (since its return value from  
          
        `fork()` is `0`) while the parent (say `pid=5`) executes lines 16–17 (since its return value is an integer > 0).