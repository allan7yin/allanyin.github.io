## The Thread Abstraction

Threads are yet another important abstraction in operating systems. So, what is a thread?

**⇒ A Thread** is a ==schedulable execution context==

- i.e it is something that we could execute on a core (the code) but also includes the context,
    - e.g. global variables, the heap, the stack, file descriptors
- recall stack contains stack frames which store local variables, return addresses, register values that need to be preserved, and temporary values
- sequetinal programs — instructions are executed in a linear or sequential order, one after the other — have one thread of execution
    - what if we wanted to run multiple tasks that a process is responsible for, at the same time on multiple cores?
        - e.g. render some text, some pictures, and execute some javascript on the webage
        - how would we implement this?

![[Screenshot_2023-10-22_at_2.03.11_PM.png]]

**Strategy:** Share what you can. Keep separate versions only if neccessary.

- each thread has its own stack and register values (e.g program counter, instruction resgiter, stack pointer, frame pointer)
- threads in the same process **share**: the code, global variables, heap, and file descriptors of the process

  

This can be seen on the left.

Let’s consider Google Chrome as an example of using multiple threads. The browser is designed with a multi-process architecture, where different tasks and components run in separate processes or threads.

- Say one thread (or process) is dedicated to executing JavaScript code. This thread’s **stack** would contain a lot of stackframes for JavaScript code
- Another thread is responsible for rendering text, then it’s **stack** would contatin a lot of stack frames for functions displaying text
- Each thread would be executing a different set of instructions so they would have their own values for the **resgisters,** e.g. program counter, the instruction register, the stack pointer, the frame pointer, return address

  

**Threads** are the most popular abstraction for concurrency. They are a _lighter-weight_ abstraction over processes, as they take less sytem resources to create and manage. In addition, they allow one process to use multiple cores in parallel. For example, threads allow a single program to overlap its I/O and computation. An example of this is a threaded webserver that services clients simultaneously:

```Python
for (;;) {
	fd = accept_client ();
	thread_create (service_client, &fd);
}											
```

Here’s a quick breakdown of the code:

- The `**accept_client**` function is likely responsible for accepting incoming client connections and assigns a file descriptor
- he `**thread_create**` function is used to create a new thread, and the `**service_client**` function is specified as the function to be executed by the new thread. The file descriptor (`**&fd**`) associated with the connected client is passed as an argument to the `**service_client**` function.

The purpose of this approach is to service multiple clients simultaneously. Instead of processing clients sequentially, a new thread is created for each client. This allows the web server to overlap I/O operations (waiting for clients to connect) with computational tasks (servicing clients) concurrently.

  

### POSIX thread API

- `int pthread_create (pthread_t *thr, pthread_attr_t *attr, void *(*fn)(void *), void *arg);`
    - creates a new thread identified by `thr`, and will execute function `fn` with arguments `arg`. `attr` is where we can provide optinal attributes such as thread stack size or use NULL for default values
- `void pthread_exit(void *return_value);`
    - destroy current thread and return a pointer to its return value
- `int pthread_join(pthread_t thread, void **return_value);`
    - waits for `thread` to exit and receives the return value
    - commonly used in situations where one thread needs to wait for the results computed on another thread.
- `void pthread_yield();`
    
    - tells the OS scheduler to run another thread or process if available
    
      
    

**→ Types of threads**

1. **User Threads** are threads that the user application creates and manages, including which one will run next
    1. Threads created and used in Java are an example of user threads
2. **Kernal Threads** are threads that the OS creates and manages, including which one will run next

  

Let’s take a moment to clarify the different betwene proesses and threads, as the can get confusing. Recall that a process is a running instance of a program, an executable file of some set of instructions. An active process has the resources the program needs to run such as:

- program registers
- program counters
- stack pointers
- memory pages

  

Rememeber that each process has its own memory address space, so one process cannot corrupt the memory space of another process. Something interesting is Chrome makes each tab a process, where once crashing tab will not affect others. So, how are processess related to threads?

  

A thread is a unit of execution within a process, where a process has **at least one thread**, called the main thread. Then, a process can have multiple threads as well. Each thread has its own **stack, registers, and program counters.** But I thought these were part of a process, not a stack. Whats going on? Consider the diagram below:

![[Screenshot_2023-10-22_at_9.31.37_PM.png]]

  

This models what we have learned about threads earlier: “threads in the same process **share**: the code, global variables, heap, and file descriptors of the process”. Since they share certain memory, they are susceptible to corrupting each other, hence why we need `context switching`, something we’ve seen before.

  

We now consider some threading models.

### Approach \#1: Kernel Threads

![[Screenshot_2023-10-23_at_9.06.53_AM.png]]

This is a one to one mapping between **kernel threads** and **user threads**. In this, each user-level thread is mapped to a separate kernel-leve thrad.

To add `pthread_create` to an OS as a kernel thread:

- start with process abstratcion in kernel but do not duplicate code, heap, global data, etc.
- `rfork` (plan 9) and `clone` (Linux) syscalls allow individual control of what to share

This approach **uses less system resoucrs** (processor time and memory) **than creating a new process** (because every thread operation must go throug the kernel (using an expensive syscall).

  

**⇒ Limitations**

- Every thread opertaion (create, exit, join, yield) must go through the kernel
    - there is a **1:1** correspondence. between user and kernel threads
    - every thread-related operation (creation, termination, context switching, etc.) requires a system call to the kernel. This involves switching from user space to kernel space, which introduces context-switching overhead
    - **Result:** thread operations are 10×–30× slower when implemented in kernel.
- One size fits all thread implementations
    - Kernel threads are desgined to cater to a waide range of application demands, which can include features like thread priorities, synchornization mechanisms, and other advanced functionality
    - This "one-size-fits-all" approach can lead to inefficiencies. For example, applications that do not require sophisticated thread features may still incur the overhead associated with a more complex and feature-rich thread implementation, impacting overall performance.
- General heavyweight memory requirements
    - kernel threads often come with certain memory requirements, such as a fixed-size stack within the kernel
    - there are also other data structures used for thread management that are designed with a heavier-weight process in mind
    - can lead to increased memory consumption for each kernel thread and ineffecient memory usage

  

### Approach \#2: User Threads

![[Screenshot_2023-10-23_at_11.34.34_AM.png]]

Let’s consider an example of this. Consider a multi-threaded web server:

- each thread handles a connection from a remote web browser
- a "fake" `**read**` function is used, making read system calls in non-blocking mode to avoid blocking the thread while waiting for data
- If no data is available, the scheduler can switch to another thread to maximize CPU utilization.

An alternative: **implement threads in a user-level library**

- many user threads, but only one kernel thread per process
    - a n:1 model
- `pthread_create` etc. are library functions accessed via function calls

  

How to implement:

- **Allocate a new stack for each** **`pthread_create`**
    - for each new thread created, a new stack is allocayted, as each thread needs its own space for local variables, function call info, etc.
- **Keep a queue of runnable threads**
    - maintain a queue of threads that are ready to exeucte (runnable)
- **replace blocking system calls (read/write/etc.) with non-blocking ones**
    - if operation would block, switch and run different thread
- **Schedule Periodic Timer Signal (setitimer):**
    - The use of a periodic timer signal, implemented using functions like `**setitimer**`, can be part of a preemption mechanism. Preemption involves interrupting the currently executing thread to allow other threads to run.

**⇒ Limitations of user-level threads in the above**

In the above, although we are multi-threading, we **cannot take advantage of multiple cores**. A blocking system will **block all threads**. Something like a page fault (page not in RAM but on disk) blocks all threads. In addition, we can have a **deadlock** (will learn more about in the future).

  

### Approach \#3: User Threads on Kernel Threads

This is an $n:m$﻿ model → **n user threads** and **m kernel threads**

![[Screenshot_2023-10-23_at_12.48.00_PM.png]]

- **user-threads** implemented on kernel threads
    - multiple **kernel-level threads** per process
    - `thread_create`, `thread_exit` still library functions as before

  

**→ Limitations**

1. **Similar Problems as n:1 Threads:**
    - The n:m threading model shares some of the same problems as the n:1 threading model, such as issues with blocked threads, deadlocks, and other synchronization problems. These are common challenges in multithreading environments.
2. **Matching Kernel Threads to Cores for Parallelism:**
    - One challenge in the n:m threading model is matching the number of kernel threads to the available CPU cores to maximize parallelism. This task is complex because:
        - The kernel needs to know the number of available cores.
        - The kernel also needs to be aware of which kernel-level threads are blocked.
        - The goal is to efficiently distribute work across available cores, but it's not always straightforward.
3. **Hiding Details from Applications for Transparency:**
    - The kernel tries to hide certain details from applications to maintain transparency. For example:
        - The kernel might manage the mapping of user-level threads to kernel threads without the application being explicitly aware of these details.
        - User-level thread schedulers may not have full visibility into the status of underlying kernel threads.
4. **Potential Mismatch Between User-Level and Kernel-Level Views:**
    - There can be a mismatch between what the user-level thread scheduler perceives and the actual state of kernel-level threads. For instance:
        - The user-level thread scheduler may think a thread is running, unaware that the corresponding kernel thread is blocked.
        - This discrepancy can lead to inefficiencies in scheduling and resource utilization.
5. **Kernel's Lack of Knowledge About Thread Importance:**
    - The kernel may not have information about the relative importance of each user-level thread. For example:
        - The kernel might preempt a kernel thread in which a user-level thread holds an important lock.
        - This lack of awareness of thread priorities at the kernel level can lead to suboptimal scheduling decisions.