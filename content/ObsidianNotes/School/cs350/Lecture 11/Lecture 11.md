Last time, we mentioned what a `spinlock` was. Now, we dive deeper into them with regards to CastorOS.

- A **spinlock** is a lock that **spins**, i.e. it repeatedly tests the lock’s availability in a loop until the lock is obtained.
- When threads uses a spinlock they **busy-wait** i.e. they use the processor while they wait for the lock.
- Spinlocks are efficient for short waiting times (a few instructions).
- They are used by CasterOS.
    - The implementation uses `atomic_swap_uint64`, which in turn is using the assembley language instruction `xchgq`, which is a variant of `xchg` that swaps quad words (i.e 64 bits)
    - _Interrupts are disabled on the processor that holds the spinlock_ and the thread will only execute a few instructions before the spinlock is released → therefore, any other threads busy-waiting for the spinlock will only spin for a short time
        - the critical section of code (the part that needs to be protected by the spinlock) is expected to be very short. The idea is to minimize the time during which the spinlock is held, reducing the likelihood that other threads will be delayed in acquiring the lock.
        - Since the critical section protected by the spinlock is short-lived, other threads waiting for the same lock won't have to spin for an extended period. This is desirable because spinning is an active waiting mechanism that consumes CPU resources. If the critical section were long, threads waiting for the lock would waste a significant amount of CPU time spinning, leading to inefficiency.  
              
            

The following code demonstrates this:

```C++
void Spinlock_Lock(Spinlock *lock) {
    /* Disable Interrupts */
    Critical_Enter();
    
    while (atomic_swap_uint64(&lock->lock, 1) == 1)
        /* Spin! */;
    
    lock->cpu = CPU();
}
```

1. `**Critical_Enter()**`**:** This function is assumed to disable interrupts. Disabling interrupts is a common technique used to create atomic sections of code in multi-threaded or multi-processor environments. This prevents preemption or interruption of the critical section.
2. `**atomic_swap_uint64(&lock->lock, 1) == 1**`**:** This line atomically swaps the contents of `**lock->lock**` with the value `**1**` and checks if the original value was `**1**`. If the original value was `**1**`, it means the spinlock was already held by another thread. In that case, the thread enters a spin-wait loop (`**/* Spin! */**`) until the lock becomes available.
3. `**lock->cpu = CPU();**`**:** Once the thread successfully acquires the lock, it records the identifier of the CPU/core that acquired the lock. This can be useful for debugging or for tracking which CPU holds the lock.

```C++
void Spinlock_Unlock(Spinlock *lock) {
    ASSERT(lock->cpu == CPU());
    
    atomic_set_uint64(&lock->lock, 0);
    
    /* Re-enable Interrupts (if not spinlocks held) */
    Critical_Exit();
}
```

1. `**ASSERT(lock->cpu == CPU())**`**:** This checks whether the CPU that is unlocking the spinlock is the same CPU that locked it. This is a sanity check to ensure that the same thread that acquired the lock is the one releasing it.
2. `**atomic_set_uint64(&lock->lock, 0)**`**:** This atomically sets the value of `**lock->lock**` to `**0**`, indicating that the lock is now released and available for other threads to acquire.
3. `**Critical_Exit()**`**:** This function is assumed to re-enable interrupts. If there are no other spinlocks held (as suggested by the comment), it's safe to re-enable interrupts at this point.

  

### Atomics and Portability

A key **_challenge_** of multithreaded code is that there are lots of **_variation in atomic instructions, consistency models, and compiler behaviour_**. Each architecture often varies in how these are implemented. Today, we have 2 praimry designs, with x64 being prevalent in servers and PC, and ARM architecture in smartphones and tablets, the issue persists due to the two platforms having different memory consistency models.

- Thankfully, C11 standard has builtin support for atomics, we just compile with GCC with the flag: `-std=c11` flag

  

### **C11 Atomics: Basics**

- C11 added the following support for sychronization
- New atomic type: e.g `_Atomic(int) foo`
    - Can wrap most basic types with `_Atomic(...)` and they become atomic.
    - All standard ops (e.g., +, −, /, ∗) become sequentially consistent.
    - Provide C function calls, e.g. `atomic_compare_exchange_strong`, to replace the need to use assembly language instructions, e.g. `cmpxchg`.
- The `**atomic_flag**` is a special type in C that provides a basic atomic operation for managing a boolean flag.
    - Unlike other atomic types, `**atomic_flag**` doesn't support direct loads and stores of its value.
    - Instead, it provides specific atomic operations for setting and clearing the flag: `**atomic_flag_test_and_set(&my_flag)**` and `**atomic_flag_clear(&my_flag)**`.
        - `**atomic_flag_test_and_set**` returns the previous value of the flag after setting it
    - Unlike other atomic types, `**atomic_flag**` must be implemented in a lock-free manner.
        - **Lock-Free:** A lock-free implementation ensures that a thread cannot be preempted (interrupted or paused) in the middle of executing the atomic instruction. This guarantees that the atomic operation is completed without interruption by other threads.
        - The intention of this is to avoid using traditional locks (instead of capure lock → critical code → release lock, we can simply check: `**atomic_flag_test_and_set(&my_flag)**` → critial if `false` is returned
- Fences also available to replace hand-coded memory barrier assembly.  
      
    

**→ Memory Ordering**

Several choices of **memory ordering** for **fences** are available:

1. `**memory_order_relaxed**`**:**
    - No specific memory ordering is required.
    - Operations can be reordered freely with respect to other memory operations.
2. `**memory_order_consume**`**:**
    - A slightly relaxed version of `**memory_order_acquire**`.
    - Typically used in scenarios where the current thread reads a value and uses it to perform further operations. The compiler and processor are allowed to reorder subsequent operations with the load, but they must respect data dependencies.
3. `**memory_order_acquire**`**:**
    - Reads and writes to memory that occur after the load cannot be reordered before it.
    - For example, if a thread acquires a lock, all the reads and writes that occur after the lock is acquired cannot be moved before the lock is acquired. This provides guarantees regarding the ordering of operations with respect to acquiring a lock.
4. `**memory_order_release**`**:**
    - Reads and writes to memory that occur before the store cannot move after it.
    - For example, if a thread releases a lock (stores a value of 0), all the writes to a variable must be completed before the lock is released. This ensures proper ordering of operations with respect to releasing a lock.
5. `**memory_order_acq_rel**`**:**
    - A combination of `**memory_order_acquire**` and `**memory_order_release**`.
    - It provides both acquisition and release semantics.
6. `**memory_order_seq_cst**`**:**
    - Full sequential consistency.
    - The most restrictive memory ordering, ensuring that all operations appear to be executed in a single, global order.
7. `**memory_order_consume**` **can always be replaced by** `**memory_order_acquire**`**:**
    - This statement suggests that in practice, `**memory_order_acquire**` is often used instead of `**memory_order_consume**`.
    - This is because some architectures and compilers may treat `**memory_order_consume**` as `**memory_order_acquire**`, and there might not be a significant practical difference between the two.

  

These are just like the fences we insertd via direct assembley code. The ones we will use are the `memory_order_relaxed`, `memory_order_acquire` to acquire a lock, `memory_order_release` for releasing a lock and `memory_order_seq_cst`.

- These memory ordering choices only apply to the thread that used them
- When using these, we need to strike a balance:
    - if the chosen model is too weak → race conditions ensue
    - if the chose model is too strong → reduced concurrency → reduced effeciency

  

Consider the following code:

```C++
_Atomic(int) packet_count;

void recv_packet(...) { 
	...
	atomic_fetch_add_explicit(&packet_count, 1,
   memory_order_relaxed);
	... 
}
```

- suppose we want to keep count of an event, say, number of packets of data that arrive ar yout Wifi card. In this case, we do not care **when** the increment happens, just as long as it **does happen.** So, we can add the `memory_order_relaxed` flag to it. But, what about situations where we do care?

  

**→ Producer and Consumer Revisited**

```C++
struct message msg_buf;
_Atomic(_Bool) msg_ready;

// producer (BETTER SOLUTION IS IN THE POINTS BELOW)
void send(struct message *m) {
    msg_buf = *m;
    atomic_thread_fence(memory_order_release); // fence
    atomic_store_explicit(&msg_ready, 1, memory_order_relaxed); // store
}

// consumer 
struct message *recv(void) {
    _Bool ready = atomic_load_explicit(&msg_ready, memory_order_relaxed); // load
    if (!ready)
        return NULL;
    atomic_thread_fence(memory_order_acquire); // fence
    return &msg_buf;
}
```

Let’s analyze them both:

- `**send**`
    - send makes a copy of the message
    - it asserts `memory_order_release` through the `atomic_thread_fence`, which means the preceeding write to `msg_buf` _**will not be reordered**_ to occur after the `atomic_store` fence. That is, reads and writes to memory that occur before the `store` and cannot be moved to after it.
    - the `atomic_thread_fence(memory_order_release)` ensures that all previous non-atomic writes and reads are completed and visible to other threads before the atomic store operation.
        
        - its worth noting, that with the above implementatoin, there is a certain risk — while the fence prevents `msg_buf` from being written to after the `atomic_store_explicit`, it does not prevent the `atomic_store_explicit` from being moved to before the write to `msg_buf`.
        - The better solution would be to include the flag inside of the `atomic_store_explicit`:
        
        ```C++
        // producer
        void send(struct message *m) {
            msg_buf = *m;
            atomic_store_explicit(&msg_ready, 1, memory_order_release); // store
        }
        ```
        
- `**recv**`
    - recv receives the message
    - the logic for this function is largely the same as the listener
    - the `atomic_load` function of `recv` sychronizes with the `atomic_store` function of `send` and receives the new value of `msg_buf` if `msg_ready` is 1 or `NULL` if there is no messge
- The `memory_order_acquire` assertion ensures that the _following read_ from `msg_buf` _will not be reordered_ to occur before the `memory_order_acquire` fence ensuring the timeliness of the message
    
    - again, this `fence` approach is not completely correct — it does not prevent `atomic_load_explicit` from being moved after the load. A better approach would be to directly add the `memory_order_acquire` flag to the load:
    
    ```C++
    // consumer 
    struct message *recv(void) {
        _Bool ready = atomic_load_explicit(&msg_ready, memory_order_acquire); // load
        if (!ready)
            return NULL;
        return &msg_buf;
    }
    ```
    
      
    
    To see another example of when we use memory ordering, consider the following spinlock implementation:
    
    ```C++
    // lock the spinlock 
    void spin_lock(atomic_flag *lock) { 
    	while(atomic_flag_test_and_set_explicit(lock, memory_order_acquire)) {}
    }
    
    // unlock the spinlock 
    void spin_unlock(atomic_flag *lock) { 
    	atomic_flag_clear_explicit(lock, memory_order_release);
    }
    ```
    
    In this example, it is like the above, where our `loads` and `stores` are directly using the memory ordering flags.
    
    - Spinlocks are similar to mutexes
- Kernels use these for small critical sections - e.g. updating the value of a variable
    
    - The calling `busy-waits` for the thread holding the spinlock to release it
    - There is no waiting (sleeping) and yielding to other threads while holding the spinlock
    
      
    

---

### Cache Coherence: the hardware view

**Coherence →** Concerns accesses to a single memory location (in the cahces of different cores), and ensures that stale (out-of-date) copies to not cause problems:

- When multiple processors are working on a shared set of data, each processor typically has its own cache. If one processor updates its local copy of the data, it's essential to propagate this update to all other processors to maintain consistency.

**Consistency →** concerns apparent ordering between multiple locations (e.g memory ordering model)

  

So, we introduce **multicore caches:**

1. **Multilevel Caches for Performance:**
    - Multicore processors employ multiple levels of caches (such as L1, L2, and sometimes L3) to address the performance gap between processor speed and main memory access time. Caches store frequently accessed data closer to the processor, reducing the need to fetch data directly from slower RAM.
        - As refresher:
            - The L1 cache is the closest to the processor core, residing directly on or very close to the CPU chip.
            - The L2 cache is located on a separate chip (although sometimes it's integrated into the same chip) and is positioned between the L1 cache and the main memory (RAM) — slower than L1, faster than RAM
            - The L3 cache is shared among all the cores in a multicore processor and is typically located on a separate chip — slower than L1, faster than RAM
2. **Opportunity for Disagreement:**
    - Having multiple caches introduces the possibility of cores (individual processing units) disagreeing about the content of a particular memory address. This is because each core may have its own copy of data stored in its cache.
3. **Bus-Based Approaches (Older):**
    - Older systems often used bus-based approaches where each core monitors a shared memory bus.
    - Snoopy protocols were employed, where each core "snoops" on the memory bus to observe transactions.
    - Write-through caching is used, meaning that when one core writes to a particular address, the other cores invalidate their copies for that address.
        - What is a **bus? Here is a quick reveiw**
            - In many early computer systems, various components, such as the CPU, memory, and peripherals, communicated with each other over a shared bus.
            - This shared bus served as a common pathway for data transfer and communication between different parts of the computer. You would have things like **data bus, address bus, control bus etc.**
            - **Snoopy Bus Protocols:**
            - a technique called "snoopy" bus protocols was often used for cache coherence. Each processor monitored or "snooped" on the shared bus to observe memory transactions.
            - When one processor wrote to a particular memory location, other processors would invalidate their copies of that memory address to maintain consistency.
4. **Limitation of Bus-Based Schemes:**
    - The problem with bus-based schemes is that they have scalability limitations. As the number of cores increases, the shared bus becomes a bottleneck, limiting the overall performance and scalability of the system.
5. **Modern Processors and Network-Based Approaches:**
    - Modern processors have shifted from bus-based approaches to network-based approaches.
    - Examples of interconnect technologies include HyperTransport (used in older AMD processors), Infinity Fabric (used in newer AMD processors), and UPI (used in Intel processors).
6. **Cache Lines:**
    - A typical cache line size is 64 bytes.
    - This size represents the amount of data transferred between different caches or main memory. It takes advantage of the principle of locality of reference, where programs tend to access data near locations they have accessed recently.
    - Caches are organized into blocks, and a cache line represents the smallest amount of data that can be stored or transferred between the main memory and the cache — a cache line is one block, these terms are used interchangeably

  

Each cahe line is **1 of 3** states. This known as **MSI Protocol**

1. `**Modified**` **(exclusive)**
    1. one cache has a valid copy of the data, and that copy is considered `**dirty**`, meaning it has been modified and needs to be written back to the main memory.
    2. Before entering this state, the cache must invalidate all other copies of the data in other caches. This is done to maintain coherence and ensure that the data is not being simultaneously modified in multiple caches. Out of date copies in other caches are called `**stale**`.
    3. The process of invalidating other copies introduces some overhead in terms of both communication and performance.
2. `**Shared**`
    1. one or more caches (and potentially the main memory) have a valid copy of the data. This state indicates that the data is unmodified and can be read by multiple caches.
    2. No invalidation of other copies is required in the Shared state, as the data is considered consistent among the caches that have it.
    3. This state is efficient for read-only operations but may require coordination when a cache wants to modify the data.
3. `**Invalid**`
    1. cache does not contain a valid copy of the data
    2. If a cache in the Invalid state needs the data, it must request it from a cache in the Modified or Shared state or retrieve it from the main memory.

  

**Fun Fact: For a cache to transition from one state to another → takes 100 (on smaller machines) to 2000 clock cycles on larger ones.**

  

![[Screenshot_2023-10-27_at_3.26.09_PM.png]]

![[Screenshot_2023-10-27_at_3.26.21_PM.png]]