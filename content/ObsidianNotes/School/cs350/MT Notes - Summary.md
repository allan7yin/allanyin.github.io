## Processes

- Operating system is layer of abstraction/intermediary between hardware and software applications
    - abstraction
    - safety
- Each process has its own **address space**, **open files**, **data**, etc.
- Functions to interact with threads is:
    - `int fork (void)`: creates new process, at sample line. Returns `0` in `child` and returns child `pid` in `parent`
    - `int waitpid(int pid, int *status, int options)`: wait for the child with pid of `pid`, `-1` for any child. Status is returned in `status`
    - `void exit (int status)`: have current process exit → `status` is `0` means sucess
    - `int kill(int pid, it sig)`: sends `sig` to process `pid` → `sig` is interrupt
    - `int execve (char *prog, char **argv, char **envp)`: replaces current running program with `prog` with arguments `argv`

## POSIX File Interface

- Functions in the POSIX file interface:
    - `open`: returns a fd (file descriptor)
    - `close`: closes the file for fd
    - `read`: copies data from file into **address space**
        - so threads share the file data
    - `write`: copies data from **address space** into file
    - `lseek`: enables non-sequential reading/writing → given byte offset, read/write to that position in the file
    - `dup2(oldfd, newfd)`: makes `newfd` a **exact** copy of `oldfd` → same fd and byte offset → closes `newfd` is already open → `-1` if error
        - whatever `oldfd` referenced before, `newfd` also references
        - often used for redirecting `stdin` and `stdout`
- `stdin`, `stdout` and `stderr` have their own default fd: `0`, `1`, `2` respectivley

## PCB (Process Control Block)

- OS maintains data about each process - in PCB
- It tracks state pf each process
    - registers, open files, other essential info etc.
- **Context Switching** is act of switching which process is in the **running** state
    - We have P0 and P1
    - P0 is interrupted or makes `syscall` → save state to PCB0 (location in PCB)
    - Load P1 data from PCB1 (idle between saving PCB0 and loading PCB1)
    - Run for while
    - P1 is interrupted or makes `syscall` → save state to PCB1
    - Load P0 data from PCB1 (idle between saving PCB1 and loading PCB0)
    - etc.

## POSIX Thread API

- The functions for this are:
    - `int pthread_create (pthread_t *thr, pthread_attr_t *attr, void *(*fn)(void *), void *arg)`: creates new thread `thr` and runs function `fn` with args `arg`
    - `void pthread_exit(void *return_value)`: destroy current thread → return value in `return_value`
    - `int pthread_join(pthread_t thread, void **return_value)`: wait for `thread` to exit and receive `return_value`
    - `void pthread_yield()`: yields current thread → scheduler runs another ready thread
- There are 3 threading models, each with pros and cons
    - **1:1**
        - expensive context-switching → switching processes every time we switch thread
        - everything requires `syscall`
    - **1:n**
        - if one thread blocks → blocks everything
        - not taking advantage of multiple cores
    - **n:m**
        - requires locking and other things to prevent issues (data races, deadlocks, etc)
        - dfficult to effeciently manage user `m` threads over `n` kernel threads → kernel does not know importance or order of user threads

## Procedure Calls

- The steps are:
    - if more than 6 args → save into registers
    - push `caller-save` registers to stack
    - push stack frame / call sub procedure
    - push `callee-save` registers
    - do stuff
    - restore `callee-save` registers
    - pop from stack frame → go back
    - restore `caller-save` registers

## Thread Switching

- Here is the order of execution:
    - scheduler decides which thread to run → `Sched_Scheduler()`
    - switch to that thread with → `Sched_switch()`
    - perform context switch → `Thread_switchArch()`
    - push callee-saved registers into `struct switchframe` → `switchstack()`
- If we we’re in hardware interrupt, instead of the first 2, we would have:
    - `struct trapframe` → create the trapframe
    - `trap_entry` → identify which device caused this
    - `interrupt handler` → handle interrupt (which will need us to swicth to thread running the handler)

## Sequential Consistency

1. Mantaining _program order_ on individual processors
2. Ensuring ==**write atomicity**==
    1. cache coherence → writes are sychronized across all memory

## Caching/Memory Models

1. **WB (Write-Back): t**his is the default caching policy. It allows writes to be initially performed in the cache, marking the portion as dirty. The actual write-back to main memory occurs when the item is about to be removed from the cache. The tradeoff is that it is more complex to implement.
2. **WT (Write-Through):** In this policy, all writes are immediately written back to main memory. The tradeoff is that writes take longer since they have to be immediately propagated to main memory.
3. **UC (Uncacheable):** This is used for device memory, where caching is not desirable. For example, when a key is pressed on the keyboard, you wouldn't want the system to read a cached value.
4. **WC (Write-Combining):** In this policy, writes are temporarily stored in a buffer and occur in a burst. This is useful for optimizing write operations to memory in certain scenarios.

## x86 Atomicity

- `lock` → makes the memory instruction atomic
    - applies for other `lock` instructions. Otherwise, can be re-ordered
- `xchg` → always a `lock` instruction. Exchanges the value stored in a register with value in main memory
- `cmpxchg` → compars 2 values and exchanges if different. Also by default `lock`
- `lfence` → load fence → all prior `load` instructions from the fence (above it) must complete before continuing to next instruction
- `sfence` → store fence → all pror `store` instructions from the fence are complete
- `mfence` → both `lfence` and `sfence` → everything above’s load and store must complete
- **atomics** and **fences** are a means of achieving sequential consistency
    - while sequential consistency can help us in multi-threading code, don’t want to depend on this memory model → not same for all machines
    - hence why we look towards locks, semaphores, cv, etc.

## Mutex (Lock)

```C++
void mutex_init(mutex_t *m, ...);
void mutex_lock(mutex_t *m);
int mutex_trylock(mutex_t *m);
void mutex_unlock(mutex_t *m);

int pthread_mutex_init(pthread_mutex_t *m, pthread_mutexattr_t attr);
int pthread_mutex_destroy(pthread_mutex_t *m);
int pthread_mutex_lock(pthread_mutex_t *m); // blocks 
int pthread_mutex_unlock(pthread_mutex_t *m);
int pthread_mutex_trylock(pthread_mutex_t *m); //  non-blocking -> tries to get mutex, if not avail, continues
```

## Condition Variables

```C++
int pthread_cond_init(pthread_cond_t *, ...);
int pthread_cond_wait(pthread_cond_t *c, pthread_mutex_t *m);
int pthread_cond_signal(pthread_cond_t *c);
int pthread_cond_broadcast(pthread_cond_t *c);
```

## Semaphores

```C++
int sem_init(sem_t *s, ..., unsigned int n);
sem_wait(sem_t *s) // originally called P()
// If the semaphore value n is greater than 0, decrement it. Otherwise, wait until n > 0 and then decrement it.

sem_signal(sem_t *s) // sem_post in pthreads, originally called V()
// Increment the value of the semaphore by 1.
```

## SpinLock

```C++
// CASTOR OS
void Spinlock_Lock(Spinlock *lock) {
    /* Disable Interrupts */
    Critical_Enter();
    
    while (atomic_swap_uint64(&lock->lock, 1) == 1) // swaps, returns true if value did not change 
        /* Spin! */;
    
    lock->cpu = CPU(); 
}

void Spinlock_Unlock(Spinlock *lock) {
    ASSERT(lock->cpu == CPU()); // checks to make sure the thread is same one that locked it in the first place 
    
    atomic_set_uint64(&lock->lock, 0);
    
    /* Re-enable Interrupts (if not spinlocks held) */
    Critical_Exit();
}

// C11
void spin_lock(atomic_flag *lock) { 
	while(atomic_flag_test_and_set_explicit(lock, memory_order_acquire)) {}
}

void spin_unlock(atomic_flag *lock) { 
	atomic_flag_clear_explicit(lock, memory_order_release);
}
```

## C11 Atomics

- The `C` library has its own atomic library, where we can specify a variable with `_Atomic(int) x`. Standard operations on this with other atomic variables are guarnateed to be atomic.
- Has its own atomic flag implementation: `atomic_flag`
    - cannot directly store and load from this → `atomic_flag_test_and_set(&my_flag)` and `atomic_flag_clear(&my_flag)`
    - must be used in **lock-free** implementation → do not capure lock → critical code → release lock, we can simply check: `**atomic_flag_test_and_set(&my_flag)**` → enters criticial if `false` is returned
- The functions we need to know:
    - **`atomic_flag_test_and_set(atomic_flag &my_flag)`****:** tests and sets the flag atomic variable to true (returns true if same, false otherwise)
        - this uses default memory ordering
        - `atomic_flag_test_and_set_explicit` will allow us to pass it a memory ordering
    - **`atomic_flag_clear(atomic_flag &my_flag)`****:** sets flag atomic variable to false
    - **`atomic_fetch_add_explicit(atomic_type *object, type operand, memory_order order)`****:** adds operand onto what object points to
    - **`atomic_store_explicit(atomic_type *object, type value, memory_order order)`**: sets value of atomic object to value
    - **`atomic_load_explicit(volatile atomic_type *object, memory_order order)`**: reads value of object
- For the above, the possible memory orderings are **(remember, these only apply to operations on atomic variables)**
    
    - `**memory_order_relaxed**`**:** No specific memory ordering is required → can be reordred freely
    
    1. `**memory_order_consume**`**:** A slightly relaxed version of `**memory_order_acquire**`.
    2. `**memory_order_acquire**`**:** Reads and writes to memory that occur after the load cannot be reordered before it.
        - For example, if a thread acquires a lock, all the reads and writes that occur after the lock is acquired cannot be moved before the lock is acquired. This provides guarantees regarding the ordering of operations with respect to acquiring a lock.
    3. `**memory_order_release**`**:** Reads and writes to memory that occur before the store cannot move after it.
        - For example, if a thread releases a lock (stores a value of 0), all the writes to a variable must be completed before the lock is released. This ensures proper ordering of operations with respect to releasing a lock.
    4. `**memory_order_acq_rel**`**:** A combination of `**memory_order_acquire**` and `**memory_order_release**`.
    5. `**memory_order_seq_cst**`**:** Full sequential consistency.
        - The most restrictive memory ordering, ensuring that all operations appear to be executed in a single, global order.

## Cache Coherence

- Need cache coherecne → cache needs to be consistent with each other after write
- Cache can differ from each other → how to solve?
- Bus (older): system bus to share memory
- can “snoop” on bus to see what is being written, and if overwrites data inside of the cache
- poor scalability
- **cache line** is amount of data transferred between different caches or main memory. Each one is either:
    
    - `**Modified**` **(exclusive)** → only this cache has this version
    - `**Shared**` → one or more caches have this copy of data
    - `**Invalid**` → no cache has this valid data
    
    ## MCS Lock
    
- We are looking to improve the **spinlocks**
- During each iteration of a spin, the thread interacts with **memory location** associated with the spinlock → via memory bus
- Each core has a `qnode` structure, here is what it looks like:
    
    ```C++
    typedef struct qnode {
        struct qnode *next;  // Pointer to the next node in the queue
        bool locked;         // A flag indicating whether the associated entity holds the lock
    } qnode;
    ```
    
- a `lock` is a pointer to a `qnode`
- `lock` is used to point to a lis of cores holding or waiting to hold the MSC lock
- `lock` will always point to the most recently pushed item of the queue
- When the `lock` is pointing to `NULL`, we know the lock is free