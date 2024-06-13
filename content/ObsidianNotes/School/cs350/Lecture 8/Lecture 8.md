So far, we learned a lot about threads. Let’s take a look at we use them in the operating system we are writing in our assignments: **castorOS**

### CastorOS: Thread Details

- Supports both kernel-leve and user-level threads
- provides basic `Pthreads` support, which is **POSIX Threading API** → see `lib/libc/posix/pthread.c`
- `Thread_KThreadCreate` is the function for creating a thread in **CastorOS** and is similar to `pthread_create`
    
    - takes a pointer to a function to run on the thread `f`
    - the functions arguments `arg`
    - and creates a **kernel-level thread** associated with that process
    
    ```Python
    Thread *Thread_KThreadCreate(void (*f)(void *), void *arg)
    ```
    
- can also create user-level threads, which the app can manage directly. The thread is associated with a **kernel thread**, which was denote with `oldThr`, with the address of the function to execute, `rip` along with the arguments `arg`

```Python
Thread *Thread_UThreadCreate(Thread *oldThr, uint64_t rip, uint64_t arg)
```

  

**→ Switching Threads**

There are 2 ways to do this. Remember, we are switching between threads of the same process:

![[Screenshot_2023-10-23_at_9.53.16_PM.png]]

We will analyze the example from above:

- Thread switches go through `Sched_Scheduler()` to decide which thread to run next
    - This will select the next thread to execute
- It calls `sched_switch()`to switch the running thread
    - transitions to the newly selected thread determined by the above
- It calls `Thread_SwitchArch`, which is customized for the particular processor that is used
    - this is architecture-specific function that performs the actual context switch between threads
- It calls `swicthstack()`. During the thread switch, the execution context switches from one thread’s stack to another. This manages the stack switching process while saving the **callee-saved** registers in a `switchframe`
    
    - `**switchframe**` **is important. This is a data structure that stores the values of callee-saved registers during the thread switch. Similar to when we call sub-procedure in that, new function will begin execution, with its own registers, variables, etc. Does not know of the existence that the registers contain data that needs to be saved, hence, the thread exiting execution needs to save to** `**switchframe**`
    
      
    
- In the case of a hardware interrupt, a `**trapframe**` is created. A trapframe is another data structure that stores the values of all registers at the time of the interrupt. This is crucial for handling interrupts and resuming the interrupted code correctly.
- After a hardware interrupt occurs, the kernel needs to identify which device generated the interrupt. This is typically done in a function like `**mgttrap_entry()**`. Once the interrupt source is identified, the appropriate interrupt handler is determined.
- Finally, the kernel switches to the code that corresponds to the device generating the interrupt. This involves changing the execution context to start running the code associated with handling the specific interrupt.
    
    - as seen above on the right stack
        - create the `**trapeframe**`
        - identity which device triggered the interrupt
        - we handle this interrupt
        - conduct context-switch by switching threads
        - we save callee-saved registers by calling `switchstack()`
        - and then construct a `switchframe` and save to this data structure
    
      
    

**Note:** In the context of computer systems, a "==**device**==" refers to any piece of hardware or peripheral that can be controlled or interacted with by the computer's operating system. Devices can include a wide range of components such as input devices (e.g., keyboards, mice), output devices (e.g., displays, printers), storage devices (e.g., hard drives, SSDs), network interfaces, and various other hardware components.

  

Consider the following code:

```Python
9 # switch(uint64_t *oldsp, uint64_t newsp)
10 # %rdi: oldsp
11 # %rsi: newsp
12 FUNC_BEGIN(switchstack)
13 # Save callee saved registers of old thread
14 pushq %rbp
15 pushq %rdi
16 pushq %rbx
17 pushq %r12
18 pushq %r13
19 pushq %r14
20 pushq %r15
21
22 # Switch stack from old to new thread
23 movq %rsp, (%rdi)
24 movq %rsi, %rsp
25
26 # Restore callee saved registers of new thread
27 popq %r15
28 popq %r14
29 popq %r13
30 popq %r12
31 popq %rbx
32 popq %rdi
33 popq %rbp
34 ret
35 FUNC_END(switchstack)
```

Here's the analysis:

- **Lines 9-11:** The function `**switchstack**` takes two parameters - a pointer to the old stack (`**oldsp**`) and the address of the new stack (`**newsp**`). These parameters are passed in registers `**%rdi**` and `**%rsi**`, respectively.
- **Lines 13-20:** Callee-saved registers (`**%rbp**`, `**%rdi**`, `**%rbx**`, `**%r12**`, `**%r13**`, `**%r14**`, `**%r15**`) of the old thread are saved onto the old stack. This is a common practice to preserve the state of the calling function across a context switch.
- **Lines 23-24:** The current stack pointer (`**%rsp**`) is saved at the address pointed to by `**%rdi**`, effectively saving the state of the old thread's stack. Then, the new stack pointer (`**%rsi**`) is loaded into `**%rsp**`, switching the stack to the new thread.
    - `(%rdi)` means don’t store it in rdi but store it at the address contained in rdi
- **Lines 27-33:** The callee-saved registers of the new thread are restored from the new stack, and the old stack's state is popped off the stack. Finally, the function returns.
- **Line 34:** `**ret**` instruction to return from the function.

  

**→ Note:** In x64, we do not need to make room on the stack by decrementing the stack poitner and copying the valuye into the stack, we need only call `push`. The same aplies for `popq`

  

**→ Ask:** Is it by chance we are popping the same registers? By chance the other thread’s callee-saved registers are the same ones as

  

### Critical Sections

Consider the following code:

```C++
int total = 0

void add() {
	for (int i = 0; i < N; i++) {
		total ++;
	}
}

void sub() {
	for (int i = 0; i < N; i++) {
		total --;
	}
}
```

What if we make 2 threads, one will run `add()` and the other will run `sub()`. What would we expect the value of **total** be after N = 100, 1000, or 10000 iterations? To examine this more closely, it actually helps to examine the assmebley pseudo-code for this:

```C++
int total = 0; /* r8 := &total */

/* Thread 1 */
void add() {
	for (int i=0; i<N; i++) {
		lw r9, 0(r8) /* total++ */
    addi r9, 1
    sw r9, 0(r8)
	} 
}

/* Thread 2 */
void sub() {
	for (int i=0; i<N; i++) {
		lw r9, 0(r8) /* total-- */
		subi r9, 1
		sw r9, 0(r8)
	}
}
```

Now, when running 2 threads, there are 3 orders we can think about in terms of execution. Interestingly, you will see how order **does matter.** For simplicty, the slides will be posted below:

  

![[Screenshot_2023-10-24_at_1.56.53_PM.png]]

![[Screenshot_2023-10-24_at_1.57.01_PM.png]]

![[Screenshot_2023-10-24_at_1.57.19_PM.png]]

  

### Need for Synchronization

- Problem: Data races occur without synchronization
- ==**Race Condition**== is when ==_**the program result depends on the order of execution**_==

To demonstrate this, consider the following code

**Program A**:

```C++
int flag1 = 0, flag2 = 0;

void p1(void *ignored) {
	flag1 = 1;
	if (!flag2) { critical_section_1(); }
}

void p2(void *ignored) {
	flag2 = 1;
	if (!flag1) { critical_section_2(); }
}

int main() {
	pthread_t tid;
	pthread_create(tid, NULL, p1, NULL); // create another thread 
	p2(); // run p2 on the current thread 
	pthread_join(tid);
}
```

So, will both threads be able to run their `critical_section` code? Consider the following orders:

- If `thread 1` runs `p1` first:
    - It will set flag1 to 1 and since flag2 is 0, it will execute `critical_section_1()`.
    - Then when thread 2 runs, it will set flag2 to 1 and since flag1 is 1, it will not execute `critical_section_2()`.
- If `thread 2` runs `p2` first:
    - It will set flag2 to 1 and sinceflag1 is 0, it will execute `critical_section_2()`.
    - Then when thread 1 runs, it will set flag1 to 1 and since flag2 is 1, it will not execute `critical_section_1()`.
- If both run at the same time:
    - both will set their flag’s to 1 and since the other flag is 1, they will not execute their critical sections.

**Wierd:** If flag2 is read before flag1 is written to RAM by one thread and flag1 is read before flag2 is written to RAM by another thread, then if there is some delay in the processors accessing RAM there will be problems.  
  

**→ Write followed by read**

This refers to a practice where a system ensures that the effects of a write operation are **visible to subsequent read operations in the program order**. In other words, the hardware ensures that when a value is written to a memory location, any subsequent read operation on that location will see the updated value. This is to prevent the above “wierd” occurrence.

- We perform this optimization in hardware: the write of each thread is buffered (i.e. stored for a few clock cycles while some other core is accessing RAM) and each thread continues to execute with out waiting for the write to complete
    - **Write Buffers:** When a write operation is performed by a thread, the updated value is stored in a write buffer or a store buffer before being written to the main memory (RAM). This allows the thread to continue executing without waiting for the write operation to be completed in the main memory.
    - **Read After Write:** When a thread performs a read operation on a memory location, it checks its own write buffer to see if there are any pending writes to the same location. If there are, the thread ensures that it reads the most recent value from its write buffer rather than directly from the main memory.
- This was fine a system with a single core and a single processor but does not work with  
    multiple cores or multiple processors.  
    
- The result will be that both threads will enter the critical section.
- ==**The system must maintain program order when a write is followed by a read.**==

  

Now, consider the following code example:

**Program B**:

```C++
int data = 0, ready = 0;

void p1(void *ignored) { 
	data = 350;
	ready = 1;
}

void p2(void *ignored) { 
	while (!ready); 
	use(data);
}
```

- When p1 runs, it will set data to 350, then set the ready flag to 1.
- When p2 runs, it will busy wait until the ready flag is 1 and then it will call use (data).
- if p1’s two writes are going to two different memory modules (say RAM and disk) and the write to `data` happens a bit slower than the write to `ready` then p2 may read the new value of ready and the old value of data.
    - Recall that the **cache line** is the unit of data transfer between the core and Level 1 cache. An optimization may be to reorder writes so that multiple write to the same cache line occur at the same time
    - When core write sdata to a specific memory location, it often involves transferring a while cache line, not the individual data
- This was fine a system with a single core and a single processor but does not work with multiple cores or multiple processors.
- The system must ==**maintain program order between two write operations**==
    - i.e the system cannot re-order write operations  
          
        

**→ Imagine the above program is written with non-blocking reads**

- A blocking read would read the value of ready first, waiting for the result, and then proceed to read data.
- A non-blocking read would execute the first read and not wait for it to be done before executing the second read.
- If p1’s two writes are going to two different memory modules and the read of `data` completes before the read from of `ready` even though it was executed later, then p2 may read the new value of ready and the old value of data.
- The system ==**must maintain program order between two read operations.**==
    - I.e. cannot speculatively prefetch data. The system must wait until `ready` is `1` before fetching `data`

  

**Program C**:

```C++
int a = 0
b = 0;

void p1(void *ignored) { 
	a = 1;
}

void p2(void *ignored) { 
	if (a == 1) {
		b = 1; 
	}
}

void p3(void *ignored) { 
	if (b == 1) {
		use(a);
	}
}
```

- If threads T1, T2, and T3 run in sequential order, we see:
    - `p1` sets `a` to `1`.
    - If `a` is `1`,then `p2` sets `b` to `1`.
    - Finally if `b` is 1, then `p3` calls `use(a)`.
- **Cache coherence** is making sure that in a multicore or multiprocessor environment, if two or more caches are storing for the same address, then the value stored for that address must be the same
    - _I.e. changes in one cache must be propagated to other caches, to maintain consistency in data version_
- A write should not be considered complete until all cached copies have been updated (or marked as invalid so that they will not be used).
- If one thread executes `p1` and updates the value of `a` and the change may propagate to `p2`’s cache before it propagates to `p3’`s cache. If `p2`’s update of `b` arrives sooner, then `p3` will see the old value of `a` and the new value of `b`.
- The updates to `a` and `b` can overlap from `T3`’s perspective. Each thread has their own view of the variables and executes their instructions in sequential order but none has the global view of the three threads and what they are each executing. Hence why it is so important to have architecture to support **synchronization.**

  

So, for the 3 code snippets above, do we every completely know what the output will be? Well, we simply _**don’t know:**_

- Such things really depend on the machine being used
- If the system provides **sequential consistency** (which not all do), then yes, we can tell what order things will run and what the output will be.
    - ==**must maintain program order between two read operations**==
    - ==**maintain program order between two write operations**==
    - ==**must maintain program order when a write is followed by a read.**==

  

### Sequential Consistency

**→ Sequential consistency** means that the result of execution is as if all operations were executed in some sequential order, and the operations of each processor occured in the in the order specified by the program. This definition boils down to 2 requirements:

1. Maintaining _program order_ on individual processors
2. Ensuring ==**write atomicity**==
    1. i.e. that if any processor reads the result of a write, then all subsequent reads by any other processor will see the same result.
    2. In other words, the write operation is guaranteed to be completed entirely, without being interrupted by other operations, and it is not divided into smaller, non-atomic parts (hence why we say “atomicity”)
    3. In the context of concurrent programming, atomicity prevents interleaving or partial execution of write operations. If a thread starts a write operation, it will be allowed to finish that operation entirely before other threads are allowed to perform any conflicting operations.
    4. This is particularly important for maintaining data consistency in shared memory environments. Without atomicity, if one thread writes to a shared variable and the write operation could be interrupted, other threads might observe an inconsistent or partial state of that variable.

  

Without sequential consistency, multiple processors can be “worse” than preemptive threads.

- May see results that cannot occur with any interleaving of threads on a single processor.
- Why doesn’t all hardware support sequential consistency?
    - 2 main reasons: **(1) sequential consistency thwarts hardware optimizations (2) sequential consistency thwarts compiler optimizations**

  

**→ SC thwarts hardware optimizations**

- **complicates write buffers**
    - recall program A from above, we would then need to read `flag2` before `flag1` or `flag` before `flag2`
- **can’t re-order overlapping write operations**
    - for example if there are concurrent writes to different memory modules or if there are coalescing writes to the same cache line, the hardware cannot optimize by reordering these writes
    - this restriction is imposed to maintain the illusions of a consistent sequential execution
- **complicates non-blocking reads**
    - Non-blocking reads, where a processor speculatively prefetches data, become complicated under sequential consistency.
    - The hardware may not be able to easily speculate and prefetch data because the order in which operations occur is critical for maintaining the sequential consistency requirement.
- **Makes cache coherence more expensive:**
    
    - Cache coherence, which ensures that multiple caches in a system have consistent views of memory, becomes more expensive under sequential consistency.
    - Write completion must be delayed until invalidation or update in certain scenarios, as mentioned in Program B. Overlapping updates are also restricted if there is no globally visible order (Program C). These restrictions add complexity and potentially reduce the efficiency of cache coherence mechanisms.
    
      
    

**→ SC thwarts compiler optimizations**

- **Code motion:** moving code around to improve performance, e.g. swap the order of the 2 add instructions to avoid load-use hazard
    
    ```C++
    lw $1, 0($30) 
    add $2, $2, $1 
    add $30, $30, $4
    ```
    
    - keeping a value in a register
        - Collapse multiple loads/stores of same address into one operation
    - Common subexpression elimination, e.g. `(n+1) * (n+1)`
        - Calculate value n+1 once and use it twice.
        - Could cause memory location to be read fewer times
- **Loop Blocking:** Re-arrange loops for better cache performance by trying to keep the inner loop entirely in cache. Rather than a single loop from 1 to 10,000, create a sequence of loops from 1 to 1000, 1001 to 2000, etc.
    - reduces number of cache misses
- **Software pipelining:** Move instructions across iterations of a loop to overlap instruction latency (keep instructions in cache) with branch cost.

```C++
for (i = 0; i<N; i++) { 
	f(i);
	g(i);
}

for (i = 0; i<N; i += 2) { 
	f(i);
	f(i+1);
	g(i);
	g(i+1);
}
```

  

[[Protection is the responsibility of the application writer (the user)]]