## x86 consistency

Memory consistency and caching models are concepts that pertain to the order and visibility of memory operations in a multi-processor system. These models define the rules and protocols governing how memory reads and writes are observed by different processors in a concurrent or parallel computing environment.

- **Memory consistency models** define the order in which memory operations (reads and writes) appear to be executed with respect to each other in a parallel or concurrent system.
- **Caching models** deal with the consistency of data stored in caches across multiple processors in a multiprocessor system. The primary goal is to ensure that each processor has a coherent view of memory, despite the use of private caches.
- In short, these models are in place to implement **sequential consistency** — something we saw in last lecture

  

So, what are some important characteristics of the **memory consistency and caching models** of the x86 archiecture:

- **Memory Type Range Registers (MTRR):**
    - MTRRs are a feature in x86 architecture that allows specifying the caching behaviour for certain ranges of physical memory. These registers are used to control the memory type (e.g. caching policy) for specific memory regions
    - Its worth mentioning that MTRR is an older method

1. **Page Attribute Table (PAT):**
    1. a newer method that allows control for each 4K page (4KiB). Provides more flexibility compared to the MTRR model.

  

Here are caching options that represent our consistency models:

1. **WB (Write-Back): t**his is the default caching policy. It allows writes to be initially performed in the cache, marking the portion as dirty. The actual write-back to main memory occurs when the item is about to be removed from the cache. The tradeoff is that it is more complex to implement.
2. **WT (Write-Through):** In this policy, all writes are immediately written back to main memory. The tradeoff is that writes take longer since they have to be immediately propagated to main memory.
3. **UC (Uncacheable):** This is used for device memory, where caching is not desirable. For example, when a key is pressed on the keyboard, you wouldn't want the system to read a cached value.
4. **WC (Write-Combining):** In this policy, writes are temporarily stored in a buffer and occur in a burst. This is useful for optimizing write operations to memory in certain scenarios.
    1. allows certain optimizations that might result in a less strictly ordered view of memory operations compared to some other memory types.

  

**Write-combining** has no caching and weak consistency (==**weak-consistency**==: no guarantee that reads and writes will have sequential consistency)

---

### x86 Atomicity

- `lock` — prefix _**makes the memory instruction that follows it atomic (cannot be interrupted)**_
    - usually locks the memory bus for duration of instruction (expensive)
    - all lock instructions totally ordered → will happen in sequential order
    - other memory instructions cannot be re-ordered with **locked** ones
    - If the memory involved in the `**lock**` operation is already exclusively cached in a single cache (meaning it's only present in one processor’s cache and not shared), it can avoid the expensive bus locking. In this case, the `**lock**` operation can be optimized without the need for locking the memory bus.
- `xchg` — exchanges the value stored in a register with the value stored in main memory
    - The `**xchg**` instruction is always considered locked, even without the explicit `**lock**` prefix. This ensures that the exchange operation is atomic.
- The `**cmpxchg**` instruction compares two values and exchanges them if they are different.
    - Similar to `**xchg**`, the `**cmpxchg**` instruction is always considered locked, even without the explicit `**lock**` prefix.

**→ We cannot re-order the relative order of the** `**lock**` **instructions, but otheres may move**

  

Now, let’s take a look at fences. In the context of assembly language instructions and memory ordering, **fence instructions** are used to control the re-ordering of memory operations in a way that ensures a desired ordering and visibility of these operations.

- `**lfence**` **(Load Fence):**
    - `**lfence**` is a load fence instruction that ensures that all prior ==**load instructions**== (reads from memory) are complete before executing the next instruction.
    - It prevents load instructions that appear before the `**lfence**` from being moved or reordered to execute after it. This guarantees that the effects of prior load operations are visible before proceeding to the next instruction.
- `**sfence**` **(Store Fence):**
    - `**sfence**` is a store fence instruction that ensures that all prior ==**store instructions**== (writes to memory) are complete before executing the next instruction.
    - It prevents store instructions that appear before the `**sfence**` from being moved or reordered to execute after it. This ensures that the effects of prior store operations are visible before proceeding to the next instruction.
    - It can be useful, for example, after non-temporal stores to data (stores that are not intended to be accessed in a cache-friendly manner) and before setting a ready flag in a program (Program B).
- `**mfence**` **(Memory Fence):**
    - `**mfence**` is a memory fence instruction that combines the functionalities of `**lfence**` and `**sfence**`.
    - It ensures that all prior reads and writes are complete before proceeding to the next instruction. This is a more comprehensive form of fencing that provides both load and store ordering.

  

**→ Assuming Sequential Consistency**

We often reason about concurrent code assuming sequential consistency (as we read lines of concurrent `c` code, and go line by line, this is how compilers work too). But, for low level code, such as assembley, we **need to know the memory model** _(sequential consistency is one such model)_

- we will need to use `atomics` and barriers in the source code to ensure that the compiler understands where there is communication between threads and where there be non-sequentially consistent behaviour if the compiler does a re-ordering operation
- For most code, we will avoid depending on the memory model
    - we accomplish this obeying certain rules (which we will see later)

  

Assume we are currently working with the sequential consistency memory model. Consider the following `prodcuer` `consumer` example with at least one thread for each. The following are some key terms:

- `buffer` stores at most `BUFFER_SIZE` ITEMS, ONE PER SLOT
- `count` is number of used slots
- `out` is the next empty buffer slot to fill (if any)
- `in` is oldest filled slot to consume (if any)

```C++
void producer(void *ignored) {
    for (;;) {
        item *nextProduced = produce_item();

        while (count == BUFFER_SIZE) {
            /* do nothing */
        }

        buffer[in] = nextProduced;
        in = (in + 1) % BUFFER_SIZE;
        count++;
    }
}

void consumer(void *ignored) {
    for (;;) {
        while (count == 0) {
            /* do nothing */
        }

        item *nextConsumed = buffer[out];
        out = (out + 1) % BUFFER_SIZE;
        count--;

        consume_item(nextConsumed);
    }
}
```

So, what can go wrong with these 2 threads? → `count` may have the wrong value and is a classic example of an issue of threads accessing shared resource. A possible implementation of `count ++` and `count —` is:

- ==**register ← count**==
- ==**register ← register + 1**==
- ==**count ← register**==

- ==**register ← count**==
- ==**register ← register - 1**==
- ==**count ← register**==

So, then a possible exection of threads could be:

- ==**register ← count**==
- ==**register ← register + 1**==
- ==**register ← count**== ==_(this overrides the above + 1)_==
- ==**register ← register - 1**==
- ==**count ← register**==
- ==**count ← register**==

  

A ==**critical section**== is the code that accesses a shared variable (or more generally, a shared resource) and must not be concurrently executed by more than one thread. The above is an example of a **data race.** But the example above needs 3 lines to increment/decrement a variable. What if we had an instruction that accomplish in one step — a single-instruction add.

- `**addl $1, _count**` on the Intel x86 (i386) architecture, is used to increment (`**++**`) or decrement (`**--**`) a variable (`**_count**`).

  

While it might seem like a concise and efficient way to perform these operations, there's a potential problem when it comes to ensuring atomicity, especially in a multiprocessor environment. This is sill not **gauranteed to be atomic**, making it susceptible to race conditions. We need a way to protect resources from being modified at once via multiple threads. We introduce some important terms now that represent the **desired outcome:**

- **Mutual Exclusion**
    - only one thread can be in critical section at a time
- **Progress**
    - if no thrad currently in the critical section and one of the treads is tryint to enter, it will eventually get in
- **Bounded waiting**
    - once a thread starts trying to enter the critical section, there is a bound on the number of times other threads get in

**Note:** progress vs. bounded waiting

- If no thread can enter the critical section, we don’t have progress.
- If thread A waiting to enter while B repeatedly leaves and re-enters ad infinitum, we don’t have bounded waiting — thread B needs to be eventually get a turn at the shared resource

  

Consider the following example — assume 2 threads $T_0$﻿ and $T_1$﻿:

```C++
// Variables
int not_turn;      // not this thread’s turn to enter
bool wants[2];     // wants[i] indicates if Ti wants to enter

// Code
for (;;) {
    /* assume i is thread number (0 or 1) */
    wants[i] = true;
    not_turn = i;

    while (wants[1 - i] && not_turn == i)
        /* other thread wants in and it is not our turn */;

    critical_section();
    wants[i] = false;
    remainder_section();
}
```

**→ This is called** ==**Peterson’s Solution**==

- **Mutual exclusion** — can’t both be in the critical section
    - would mean `wants[0] == wants[1] == true`, so `not_turn` would have blocked one thread from entering the critical section
    - as seen, even if `wants[1] == 1`, since we have `not_turn == 1`, we cannot enter the critical section
- **Progress** — If $T_{i-1}$﻿ not in critical section, it can’t block $T_i$﻿
    - Means `wants[i-1] == false`, so $T_i$﻿ won’t loop
- **Bounded waiting** — similar argument to progress
    - if $T_i$﻿ wants lock and $T_{i-1}$﻿ tries to re-enter, $T_{i-1}$﻿ will set `not_turn = 1 - i`, allowing $T_i$﻿ in

  

This solution is **very expensive** as it used busy waiting — busy waiting refers to a situation where a process (or thread) continuously checks a condition in a loop without yielding the processor to other tasks. While we can generalize this to n threads, this solution is not ideal.

- We will need to adapt the machine memory model if not **sequentially consistent**, as peterson’s solution makes many assumptions from the sequential model
    
    - In particular, it assumes that writes to memory by one processor become visible to other processors in a well-defined order.
    - Ideally, we want our code to be able to run everywhere, regardless of memory model
    
      
    

Ideally, we want to abstract away the underlying details of concurrent progrmaming and synchornization away programmer. To do so, we introduce mutex’s.

---

### Mutex

Mutexes, short for "mutual exclusion," are synchronization primitives commonly used in concurrent programming to manage access to shared resources. The primary purpose of a mutex is to ensure that only one thread can access a critical section of code or shared resource at any given time. This prevents data races and ensures the integrity of shared data.

Thread packages typcially provide `mutexes`:

```C++
void mutex_init(mutex_t *m, ...);
void mutex_lock(mutex_t *m);
int mutex_trylock(mutex_t *m);
void mutex_unlock(mutex_t *m);
```

Only one thread can acquire `m` at a time, others wait. So, how can we use this?

- **All global data should be protected by a mutex**
    
    - Global = accessed by more than one thread, at least one write
    - The execption is initialization, before exposed to other threads
    
    [[Protection is the responsibility of the application writer (the user)]]
    
- **Compiler/Runtime Contract (C, Java, Go, etc.):** assuming no data races, the program behaves sequentially consistent
- If you use mutexes properly, behaviour should be indistinguishable from sequential consistency
    - responsibility of threads package and compiler
    - mutex is broken if you use properly and don’t see sequential consistency
- OS Kernels also need sychronization
    - some mechanisms like mutexes
    - but interrupts complicate things (imcompatible with mutexes)

  

**→** Now, we show the **PThread Mutex API:**

```C++
int pthread_mutex_init(pthread_mutex_t *m, pthread_mutexattr_t attr);
// Initialize a mutex
// m is a pointer to the mutex object 
// attr are Mutex attributes, can specify attributes such as type of mutex (normal, error-checking, recursive)

// returns 0 on success, non-zero on failure 

int pthread_mutex_destroy(pthread_mutex_t *m);
// Destroy a mutex
// m is a pointer to the mutex object 

// should destory a mutex as soon as its no longer needed to free associated resources 

int pthread_mutex_lock(pthread_mutex_t *m);
// Acquire a mutex
// m is a pointer to the mutex object 

// If the mutex is already locked by another thread, the calling thread may be blocked until the mutex becomes available.
// this is a blocking operation, blocks calling thread if mutex is unavailable

int pthread_mutex_unlock(pthread_mutex_t *m);
// Release a mutex
// m is a pointer to the mutex object, should be unlocked after 

int pthread_mutex_trylock(pthread_mutex_t *m);
// Attempt to acquire a mutex
// m is a pointer to the mutex object 

// this is a non-blocking operation, DOES NOT block calling thread if mutex is unavailable

// Return 0 if successful, otherwise -1 (errno == EBUSY)
```

With these locking mechanisms in place, we take a look at the sample `consumer` and `producer` code improved:

**Producer:**

```C++
mutex_t mutex = MUTEX_INITIALIZER;

void producer(void *ignored) {
    for (;;) {
        item *nextProduced = produce_item();
        mutex_lock(&mutex);

        while (count == BUFFER_SIZE) {
            mutex_unlock(&mutex);  /* <--- Allowing other threads to access the mutex */
            thread_yield(); // yields the processor to allow other threads to run
            mutex_lock(&mutex); // re-acquire the mutex, remember this is a blocking call 
        }

				// the pattern above, of releasing then capturing, avoids busy-waiting 

        buffer[in] = nextProduced;
        in = (in + 1) % BUFFER_SIZE;
        count++;

        mutex_unlock(&mutex);
    }
}
```

**Consumer:**

```C++
void consumer(void *ignored) {
    for (;;) {
        mutex_lock(&mutex);

        while (count == 0) {
            mutex_unlock(&mutex);
            thread_yield(); // yields the processor to allow other threads to run
            mutex_lock(&mutex);
        }

        item *nextConsumed = buffer[out];
        out = (out + 1) % BUFFER_SIZE;
        count--;

        mutex_unlock(&mutex);

        consume_item(nextConsumed);
    }
}
```

**Notes:**

- The global variables `count`, `in`, `out` and `buffer` are accessed by the producer and the consumer, so they should be protected by a mutex.
- If `count` is `0`, then release the mutex and call `thread_yield` to allow another thread to run.
- Acquire mutex again to check the value of `count` and possibly access the buffer.
- While waiting for the mutex the thread does not use the processor but instead waits in a wait **queue** and the kernel wakes it up when the other thread has unlocked the the mutex.
- However each thread is continually checking the value of count, releasing the mutex, yielding, and tryto to lock the mutex again, which does use up processor time. Upon resuming from blocking stage in `mutex_lock(&mutex);`, `count` may not have changed, and again, we go through the process of changing threads. This can
- Busy-waiting in an application is inefficient. The above, while helps reduce it, still has needless processing in that loop.
    - Thread consumes processor time even when can’t make progress.
    - This approach unnecessarily slows other threads and processes.

  

A solution is to implement **condition variables**, to tell the thread scheduler which threads are available to be run. To do so:

```C++
// Initialize with specific attributes
int pthread_cond_init(pthread_cond_t *, ...);

// Atomically unlock m and put the thread to sleep until c signaled
// Then re-acquire m and resume executing
int pthread_cond_wait(pthread_cond_t *c, pthread_mutex_t *m);

// Wake one thread waiting on c, first one in queue
int pthread_cond_signal(pthread_cond_t *c);

// Wake all threads waiting on c
int pthread_cond_broadcast(pthread_cond_t *c);
```

- Condition variables get their name because they allow threads to wait for arbitrary conditions to become true inside of a critical section.
- Normally, each condition variable corresponds to a particular condition that is of interest to an application.
- For our `producer` and `consumer` example from above, we could use 2 **condition variables:**
    - `count` > 0 (items in the buffer)
    - `count` < 0 (there is free space in the buffer)
- When a condition is not true, a thread can `cond_wait` on the corresponding condition variable (as it will be notified when the condition is ready), unlocking the mutex until the condition becomes true.

  

- When another thread detects that a condition is true, it uses `cond_signal` or `cond_broadcast` to notify any blocked threads.
    - Note that signalling (or broadcasting to) condition variable that has no blocked threads has no effect. ⇒ Signals do not accumulate.  
          
        

With this, we can again, improve our `consumer` and `producer` code:

**Producer:**

```C++
mutex_t mutex = MUTEX_INITIALIZER;
cond_t nonempty = COND_INITIALIZER;
cond_t nonfull = COND_INITIALIZER;

void producer(void *ignored) {
    for (;;) {
        item *nextProduced = produce_item();
        mutex_lock(&mutex);
        while (count == BUFFER_SIZE)
            cond_wait(&nonfull, &mutex); // we always want to re-check the condition upon wake up, hence the while loop
        buffer[in] = nextProduced;
        in = (in + 1) % BUFFER_SIZE;
        count++;
        cond_signal(&nonempty);
        mutex_unlock(&mutex);
    }
}
```

**Consumer:**

```C++
void consumer(void *ignored) {
    for (;;) {
        mutex_lock(&mutex);
        while (count == 0)
            cond_wait(&nonempty, &mutex);

        item *nextConsumed = buffer[out];
        out = (out + 1) % BUFFER_SIZE;
        count--;

        cond_signal(&nonfull);
        mutex_unlock(&mutex);

        consume_item(nextConsumed);
    }
}
```

![[Screenshot_2023-10-25_at_11.16.50_PM.png]]