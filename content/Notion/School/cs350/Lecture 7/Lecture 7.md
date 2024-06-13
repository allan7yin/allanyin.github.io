Last lecture, we looked at threads, and the different ways we could use them, whether that be 1:1, n:1, or n:m thread models. Continuing from there, consider this small case study:

### Case Study: Go Language and Go Routines

- Go routines (a.k.a. goroutines) are very light-weight user-level threads. Lighter than something like `Java Thread`, as those can get quite resource hungry.
- Go routines run on top of kernel threads (n:m Model)
    - multi-core scalability and efficient user-level threads
    - one pthread (kernel-level thread) per core
    - supports as many user-level threads as you would like
- Every logical core is owned by a kernel thread when running.
    - Each kernel-level thread finds and runs a go routine (user-level thread)
- Convert blocking system calls (when possible):
    - Converted to non-blocking by yielding the core to another go routine.
    - Cores poll using kernel event API poll for files, epoll for network connections, or  
        kqueue to check whether an event has happended or not.  
        
- Compare this approach with blocking system calls:
    
    - Release the core to another kernel-level thread before the call (which is expensive).
    - Let the kernel-level thread sleep (move to waiting state).
    - Move to the kernel-level thread to the ready queue when the system call has been  
        completed.  
        
    
      
    

**Go routines communicate and synchronize through channels —** a channel allows one goroutine to send data to another goroutine. It provides a safe and synchronized way for goroutines to exchange information. It has FIFO behaviouor (like a queue), the first value send into the channel is the first value that will be received.

- Channels have blocking behaviour. When a goroutine sends data on a channel, it will block until another goroutine is ready to receive the data. Similarly, if a goroutine attempts to receive data from an empty channel, it will block until another goroutine sends data.

  

### Registers and Processers

We step away for threads for a brief moment to recount some knowledge from cs241. In cs350, we largely need the same knowledge, but the teminology/notation used is slightly different:

- **Processor:** we will use an Intel x86-64 processor instead of **MIPS**
- **Register Names:** we start register names with `%` rather than `$` like we did in MIPS
- **Register Names:** also, we named the registers $0, $1, … In cs350, we refer to many of them by their default function: `%rsp` (for stack pointer)
- **Passing Arguments:** in cs241, we passed all arguments on the stack. In cs350, we pass the first 6 args in registers and if there are more than 6 args, we will pass the rest on stack
- **Saving Registers:** in cs241, the callee saved most of the registers (except frame pointer and return address). Shortcoming is we may be saving registers that caller no longer needs

  

There are typically 2 types of registers:

1. ==**caller-save**== registers are not preserved across subroutine calls, i.e. if the caller has a value in one of these registers that it wants preserved, it should store the value on the stack before calling any subroutines and restore it afterwards otherwise the callee may change its value.
2. ==**callee-save**== registers are preserved across subroutine calls, i.e. if the subroutine uses one of these registers, it must first store its current value on the stack and restore it before returning.  
      
    

x86 has some **different terminology compared to MIPS processors** and only have 16 registers. Some of the names are based on conventional usage:

**→ General Pupose Registers:**

- `%rax`, accumulator register
- `%rbx`, base pointer for memory access, e.g. an array
- `%rcx`, counter register for loops
- `%rdx`, data register for I/O and arithmetic
- `%rdi`, destination index (for copying data)
- `%rsi`, source index (for copying data)
- `%r8 – %r15`

  

**Interesting:** _while the above have particular uses in their names, they are from historical reasons. So, while they might have had more specific roles in the past, in modern x86-64 assembly,_ _`**%rdi**`_ _and_ _`**%rsi**`_ _are generally treated as general-purpose registers used for passing parameters. The naming conventions have persisted more out of tradition than strict adherence to the original source/destination indexing roles._

  

**→ Non-General Purpose Registers**

- `%rbp`, base pointer (a.k.a. frame pointer)
- `%rsp`, stack pointer
- `%rip`, instruction pointer (a.k.a. program counter)

  

Now, we will mention some conventions:

![[Screenshot_2023-10-23_at_5.36.21_PM.png]]

- for subroutine calls, the first 6 args go into `%rdi`, `%rsi``%rdx``%rcx``%r8`, and `%r9` respectively.
    - return value goes in `%rax` and `%rdx`
- The **caller-saved** registers are:
    - the args and the return registers
    - `%r10` and `%r11`
- the callee-saved registers are `%rbx`, `%rbp` and `%r12 – %r15`.
- `%rsp` is the stack pointer.
- `%rbp` is the base pointer (a.k.a. frame pointer).
- local variables are stored in registers and on stack

  

**→ Background**

The last part of this lecture, we will go over some quick review over procedure calls.

![[Screenshot_2023-10-23_at_6.25.54_PM.png]]

**→ Threads vs Procedures**

1. **Order of Resumption:**
    - **Threads:** Threads may resume execution out of order. Thread execution is not strictly Last In, First Out (LIFO). This means that the order in which threads are scheduled to run may not follow a predictable LIFO pattern.
    - **Procedures:** Procedure calls and returns follow a LIFO order. When a function is called, it gets added to the call stack, and when the function returns, it is removed from the stack in a Last In, First Out manner.
2. **Stacks for State Saving:**
    - **Threads:** Each thread has its own stack. This is because threads may not resume in a predictable LIFO order, and having a separate stack for each thread allows for independent management of the execution state.
    - **Procedures:** In traditional function calls, the LIFO nature of the stack suffices for saving and restoring the state during function calls and returns.
3. **Procedure Calls vs. Thread Switches:**
    - **Procedure Calls:** Procedures (functions) are called more often than threads switch. It's common for a thread to make multiple procedure calls during its execution.
    - **Threads:** Thread switches are less frequent compared to procedure calls. A single thread can make many procedure calls before a thread switch occurs.
4. **Register Partitioning:**
    - **Don't Partition Registers for Threads:** Unlike procedures, you generally don't partition registers for threads. This is because you may have many threads, and a unified set of registers allows for more flexibility and adaptability in managing the execution state of threads.
5. **Involuntary Interruption:**
    - **Synchronous Interruption (Procedure Call):** When a procedure is called, the compiler saves the required state in the stack. This is part of the normal procedure call mechanism.
    - **Asynchronous Interruption (Thread Switch):** When a thread switch occurs, all registers are saved. This is necessary to ensure that the state of the executing thread is preserved before switching to another thread.
6. **Concurrency:**
    - **Procedure Calls:** Only one procedure runs at a time. The scheduling of procedure calls is straightforward – the called procedure runs.
    - **Threads:** More than one thread can run at a time. The challenge in thread scheduling is determining what to run next and on which core.