In this lecture, we look to improve spinlocks. `Test-and-set` spinlock has several advantages:

1. **Simplicity:**
    - The test-and-set spinlock is straightforward to implement and understand. It involves a simple atomic operation for setting a memory flag, making it accessible for various processors and cores.
2. **Ease of Management:**
    - It is easy to manage because it utilizes a single memory location for coordinating the lock state, making it suitable for synchronization across multiple processors and cores.

Recall the example spinlock from last lecture:

```C++
// lock the spinlock 
void spin_lock(atomic_flag *lock) { 
	while(atomic_flag_test_and_set_explicit(lock, memory_order_acquire)) {} // keep iteratin until free -- busy wait 
}

// unlock the spinlock 
void spin_unlock(atomic_flag *lock) { 
	atomic_flag_clear_explicit(lock, memory_order_release);
}
```

But, this approach also has its disadvantages:

1. **High Memory Bus Traffic:**
    - One significant drawback is the potential for high traffic over the memory bus, especially when multiple threads are actively spinning. This can lead to congestion and reduced overall system performance.
2. **Fairness Concerns:**
    - The test-and-set spinlock may not guarantee fairness, particularly on the same core. A thread running on the same core can repeatedly acquire the lock, potentially leading to unfairness in resource access.
3. **Even Less Fair on NUMA Machines:**
    - Non-Uniform Memory Access (NUMA) architectures can exacerbate fairness issues, as accessing remote memory can introduce additional latency and contention.
        - **Non-uniform memory access (NUMA)** each processor has local memory and can access its memory faster that memory local to another processor or shared between processors.
            - Just your standard multi-core computer

So, how can we address these issues?

1. Avoid **spinlocks** all together, and instead, use lock-free algorithms
2. _Reduce bus traffic with better spinlocks_ — design lock that spins only on local memory, which also gives better fairness
    1. the idea is to design the spinlock in such a way that, when a thread is waiting for the lock to become available, it primarily monitors and waits for changes in the state of the lock in its own local memory.
    2. Traditional spinlocks might involve frequent communication over the system's memory bus as threads contend for the lock. By spinning only on local memory, the design aims to minimize the need for extensive communication with other cores or processors over the shared memory bus.

---

### MCS Lock

The idea is to improve upon the spinlock idea. To quickly review, how do spinlocks use the **memory bus?** Spinlocks use the memory bus to communicate and coordinate between different cores or processors in a multi-core system. The memory bus is a communication pathway that allows data to be transferred between the CPU, memory, and other components. In the context of spinlocks, the memory bus is crucial for ensuring that threads on different cores can coordinate their access to shared resources.

- During each iteration of the spin, the thread interacts with the **memory location associated with the spinlock**, potentially leading to memory bus activity.
- In a scenario where multiple threads contend for the same spinlock, there can be increased contention on the **memory bus**. This contention arises as threads on different cores compete to read and write to the memory location associated with the spinlock.
- Once a thread obtains the spinlock, it must write to it to mark it as “locked”, which again interacts with the **memory bus**
- When ensuring cache coherence — Memory bus transactions may involve signaling other cores about changes to the shared memory (recall snooping protocols — each cache "snoops" or monitors the memory bus for transactions involving the addresses of its cached data.)

  

**→ we are looking to reduce bus trafic on cache-cohrent machines and improve fairness**

Each processor has a `qnode` structure in local memory, which is fine as we know every user-thread will be mapped to kernel thread, which then runs on a processor. The purpose of a `**qnode**` is to represent a processor in a queue associated with a synchronization operation. Here is what it looks like:

```C++
typedef struct qnode {
    struct qnode *next;  // Pointer to the next node in the queue
    bool locked;         // A flag indicating whether the associated entity holds the lock
} qnode;
```

- **note:** local can mean local memory in NUMA machine or just its own cache line that gets cached in exclusive mode.
- The cache line containing the data structures related to the spinlock (such as the `**qnode**` structure mentioned earlier) is marked as exclusive to that processor. In an exclusive cache line, the processor has the sole right to modify the data without interference from other processors.

  

In MCS locks:

- A `lock` is a pointer to a `qnode` → `typedef qnode *lock`
- a `lock` is used to point to a list of processors holding or waiting to hold the MSC (spin) lock (as mentioned above)
- In this design, the `lock` will always point to the most recently pushed item of the queue.
- When the `lock` is pointing to `NULL`, that is: `*lock == NULL`, then we know the lock is free
- When a thread tries to acquire the lock, it will add itself to the list

  

Here is the code for MCS Acquire:

```C++
1   acquire(lock *L, qnode *I) {
2       I->next = NULL;
3       qnode *predecessor = I;
4       XCHG(predecessor, *L); /* atomic swap */
5       if (predecessor != NULL) {
6           I->locked = true;
7           predecessor->next = I;
8           while (I->locked)
9               /* spin */;
10  }
```

We will analyze the above code line by line for 2 different situations:

1. when the queue is empty
2. when there are already nodes queue inside of the queue

  

**→ Situation 1**

Consider the following instance:

(Ignore the `locked = 0` for the `predecessor`, not relevant for it, only the nodes in the queue need it)

![[Screenshot_2023-10-28_at_5.26.20_PM.png]]

So, we’re starting in a state where the `lock` is pointing to `NULL` and we as per lines `1-3`. Then, in line `4`, we perform an atomic exchange between the `lock` and `predecessor`, meaning we get the resulting structure:

![[Screenshot_2023-10-28_at_5.26.42_PM.png]]

So, in lines `5-onwards`, we check whether the `predecessor` pointer is `NULL`, basically checking if the queue is empty. In this case, yes, the `predecessor` pointer is `NULL`, meaning we do not need to spin/wait for the lock, it is already ready. So, we do not enter the `if` statement.

  

**→ Situation 2**

Now, what if the queue is not empty when we add `I` to the queue? Let `H` be the node that is either currently holding the `lock` or next in line to hold the `lock`. Consider the following:

![[Screenshot_2023-10-28_at_5.32.48_PM.png]]

Again, as per lines `1-3`, we perform an atomic exchange:

![[Screenshot_2023-10-28_at_5.35.00_PM.png]]

Now, we check if `predecessor` is `NULL`, which it is not, as it points to `node H`. So, we enter the loop:

- So, we set `locked = 1` for `node I`
- then, we set `predecessor->next = I`, which makes `node H` then point to `node I`
- then, we spin until `node H` releases the lock

The final structure is:

![[Screenshot_2023-10-28_at_5.40.15_PM.png]]

  

**Similarly, let’s look at the code for MCS Release**. Consider the following code:

```C++
1   release(lock *L, qnode *I) {
2       if (!I->next).              // I is the last node in the queue
3           if (CAS(*L, I, NULL))   // atomic compare and swap -> swaps if the Lock is pointing to I, makes Lock point to NULL
4               return;             // this means we are at the end of the queue, empty now -> make L point to NULL
5       while (!I->next) /* spin */; 
6       I->next->locked = false;
7   }
```

This code is fairly straightforward to understand. Consider the diagram from above, and we are trying to release the lock on `node H`. So, first, we check if `H->next` is `NULL`, which it is not, as it points to `node I`. So, we also skip the `spin` loop and set `H->next-locked = false`, which tells the processor for `node I`, which is likely spinning waiting for `I->locked = true`, that it can now exit the loop and obtain the lock.

  

Why do we `spin` in the above? Recall this from above:

![[Screenshot_2023-10-28_at_5.35.00_PM.png]]

So, in the above, say we are trying to release `node H`, while the above operation is trying to acquire the lock for `node I`. During the process of `node I`, the queue will look like the above. But now, if we were to run `release` on the above state, we would observe:

![[Screenshot_2023-10-28_at_5.40.15_PM.png]]

- `H->next == NULL`, so we would enter the first `if` statement
- But, `L` points to `node I` which is not equal to `node H`, which is what we are trying to release
- In this case, we would not return, and the `lock` would not be set to `NULL` since the comparison failed
- Now, we then enter `while` loop and spin, waiting until the acquiring code is finished, where we get rightside. Then, we can proceed.

### DeadLock

Now, we move onto a very important problem in the context of multi-threading programs. Consider the following code:

  

```C++
mutex_t m1, m2;

void f1(void *ignored) {
    lock(m1);
    lock(m2);
    /* critical section */
    unlock(m2);
    unlock(m1);
}

void f2(void *ignored) {
    lock(m2);
    lock(m1);
    /* critical section */
    unlock(m1);
    unlock(m2);
}
```

It is dangerous to acquire locks in different orders: if `T1` executes `f1` exactly when `T2` executes `f2` then `T1` is wait for `T2` to release `m2` and `T2` will wait for `T1` `m1`. This situation is called **deadlock**, i.e. both threads will wait for ever.

  

For another example, consider the following:

```C++
mutex_t a, b;
cond_t c;
ready = false

// T1
lock(a);
lock(b);
while (!ready)
    wait(b, c); // releases b but not a
unlock(b);
unlock(a);


// T2
lock(a);
lock(b);
ready = true;
signal(c); // needs a to signal c
unlock(b);
unlock(a);
```

- in `T1`, we lock `a` and lock `b`. Then, while `ready` is `false` we wait. So, we put the thread to sleep, which releases the lock on `b` and will regain the lock to `b` when the thread wakes up → a.k.a the condition `c` becomes true.
- Heres’ the issue. In `T2`, we are now trying to lock `a`, but we are blocked. `a` has not been released by `T1`, so now, both threads cannot progress and are waiting on the other — **deadloc**k
    - The potential danger arises when a thread holds a lock and calls into a lower-level function that also needs the same lock. If not handled carefully, this can lead to deadlock or other synchronization issues.
        - In this case, kind of like `t1` runs, then calls functinon (`T2`) which needs the same lock

  

**→ Deadlock Conditions**

What conditions cause deadlocks?

1. **Mutual Exclusion (Limited access):**
    - Resource can only be shared with finite users
2. **No Preemption:**
    - Resources cannot be forcibly taken away from a process; they must be released voluntarily. If a process holds a resource and requests additional resources, it may have to wait indefinitely, leading to potential deadlock.
3. **Hold and Wait (Multiple independent requests):**
    - Processes must be allowed to request resources incrementally and can hold resources while waiting for additional ones (so once acquired a lock, can keep it while waiting for another lock). If a process is holding a resource and waiting for another, and other processes are doing the same, it can result in circular waiting and potential deadlock.
4. **Circular Wait:**
    
    - A circular chain of processes, each holding a resource needed by the next process in the chain, must exist. This circular waiting scenario completes the set of conditions necessary for deadlock
    
      
    

So, how do we combat this? 2 solutions → **(1) Pro-active: prevention (2) Reactive: detection + corrective action**

**→ Pro-active: prevention**

To do this, we simply need to invalidate one of the 4 conditions mentioned above:

1. **Mutual Exclusion (Limited access):**
    1. Buy more resources, split into pieces, or virtualize to make "infinite" copies
2. **No preemption:**
    1. Physical memory: virtualized with VM, can take physical page away and give to another process!
3. **Hold and Wait (Multiple independent requests):**
    1. To prevent the "Hold and Wait" condition, a process must request all the resources it needs at once. This approach ensures that a process won't hold any resources while waiting for additional ones. However, this requires processes to know their resource needs in advance, which may not always be practical.
4. ==**Circular Wait (most common method to prevent deadlock):**==
    1. Circular waiting is prevented by using a single lock for the entire system or by establishing a partial ordering of resources. If there is a clear order in which resources must be acquired, circular waits are less likely to occur.
        1. e.g. make sure order of locks being acquired is always the same order

  

- View system as graph
    - Processes, threads, and resources are nodes
    - Resource Requests and Assignments are edges
- If graph has no cycles → no deadlock
- If graph contains a cycle
    - Definitely deadlock if only one instance per resource type, e.g. one mutex m1.
    - Otherwise, maybe deadlock, maybe not, e.g. multiple pages in RAM.
- Prevent deadlock with partial order on resources
    - E.g., always **acquire mutexes in a specific order, e.g. m1 before m2**
        - does not scale well into larger code bases
    - Statically assert (decide on a) lock ordering for the whole code base (e.g., VMware ESX)
    - Dynamically find potential deadlocks [Witness]

  

So now, let’s look at one last thing. We look at how the OS implements these higher-level locking primatives — mutexes, semaphores, condition variables. The main thing we use is something called **wait channels.**

---

### Wait Channels

What are wait channels? Wait channels are a mechanism for managing sleeping threads, especially the context of sychronization primitives like mutexes, semaphors, and condition variables. We can imagine that the **wait channel** is an object that a thread can **subscribe to and go to sleeep**. When the wait channel gets activated, any threads that are waiting on that wait channel get woken up and put in the ready queue. The following are 4 important functions that make up the `wait channel` abstraction.

```C++
void WaitChannel_Sleep(WaitChannel *wc);
void WaitChannel_WakeAll(WaitChannel *wc);
void WaitChannel_Wake(WaitChannel *wc);
void WaitChannel_Lock(WaitChannel *wc);
```

The functionality is similar to what we’ve seen for condition variables. So, what does each one do?

- `**WaitChannel_Sleep**`
    - **_Blocks calling thread_** on wait channel `wc` i.e. puts the thread to sleep
    - The current thread subscribes to this channel → imagine that a channel maintains a list of threads
- `WaitChannel_WakeAll`
    - _**Unblocks all threads**_ sleeping on the wait channel → every thread subscribed to `wc` is now awake
- `WaitChannel_Wake`
    - _**Unblocks one thread**_ sleeping on the wait channel
- `WaitChannel_Lock`
    - This is what makes using **wait channels** important.
    - _**Lock wait channel**_ for sleep and wake operations
        - Once this is called, following calls to `sleep` and `wake` will not work
        - This functionality prevents a race condition between sleep and wake operations

  

This kind of design choice is called **hand-over-hand locking —** allows for fine-grained locking, i.e. many locks. The basic idea behind hand-over-hand locking is to have threads or processes acquire and release locks in a ==**structured and sequential manner**==. Each thread holds a lock for a short duration, performs its operation, and then passes the lock to the next thread in a predetermined order.

- This is useful for concurrent access to a single data structure
    - each thread holds at most 2 locks: the previous lock and the next one: working your way through a sequence of steps
    - The locks must be ordered in a specific sequence to avoid deadlocks and ensure proper coordination among threads. This ordering is essential for maintaining consistency in the access to the data structure.
    - no going backwards — Once a thread acquires a lock for a specific element and moves forward, it should not go back to a previous element and acquire its lock again.
        - once a thread has acquired a lock, performed its operations, and released the lock, it should not reacquire the lock for a previous resource in the sequence:
            1. Thread T acquires and uses Lock 1, modifying some resource.
            2. After completing its work, Thread T releases Lock 1.
            3. Thread T then acquires Lock 2, works with it, and releases it
            4. Thread T should not then acquire Lock 1 again, must sleep and let other threads acquire
    - Going backward while holding locks could lead to potential issues, including the risk of deadlocks. By moving only forward in the sequence, the algorithm ensures a clear and consistent order of lock acquisition, reducing the likelihood of deadlocks.

Here is a quick example:

```C++
lock(A)
// Operate on A
lock(B)

// I now hold 2 locks 
unlock(A) // cannot reacquire the lock for A 

// hold 1 lock now 
// Operate on B
lock(C)

// hold 2 locks 
unlock(B)

// hold 1 lock
// Operate on C
unlock(C)

// hold 0 locks 
```

Let’s see how we can implement **semaphore** functionality using `Wait Channel` functions we showed above. Mutexes, CVs and Semaphores use **wait channels** and **hand-over-hand** locking. Consider the following code:

```C++

typedef struct Semaphore {
    int sem_count;
    Spinlock *sem_lock;       // exclusive access to this struct’s fields
    WaitChannel *sem_wchan;   // queue where blocked threads wait (i.e. sleep)
} Semaphore;

Semaphore_Wait(Semaphore *sem) {
    Spinlock_Lock(&sem->sem_lock);
    
    while (sem->sem_count == 0) {
        // Locking the wchan prevents a race on sleep
        WaitChannel_Lock(sem->sem_wchan);
        // Release spinlock before sleeping
        Spinlock_Unlock(&sem->sem_lock);        
        // Wait channel protected by its own lock, when sleeping the wait channel, its lock is released 
        WaitChannel_Sleep(sem->sem_wchan);        
        // Recheck condition, no locks held
        Spinlock_Lock(&sem->sem_lock);
    }
    
    sem->sem_count--;
    Spinlock_Unlock(&sem->sem_lock);
}

Semaphore_Post(Semaphore *sem) {
    Spinlock_Lock(&sem->sem_lock);
    
    sem->sem_count++;
    WaitChannel_Wake(sem->sem_wchan);
    
    Spinlock_Unlock(&sem->sem_lock);
}
```

**→ Question asked for clarification on Piazza**