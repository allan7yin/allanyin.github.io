### Thrashing

- **Thrashing** is when an application is _constantly swapping pages in and out_ **(between RAM and disk)** due to page faults_,_ preventing it from making progress at a reasonable rate.
- **Cause:** processes require more memory than system has → so constantly need to evict and swap pages to page table
    - Each time one page is brought in, another page, whose contents will soon be referenced (so will soon be swapped back in), is swapped out
    - Processes will spend much of their time blocked, waiting for pages to be fetched from disk
    - Disk will be very close to 100% utilization, but the system is not making much progress
- **What we wanted:** more virtual memory than installed RAM with access time the speed of primary memory

  

**→ Reasons for Thrashing:**

- The memory access pattern does not exhibit temporal locality (past $\neq$﻿ future)
    - Recall temporal locality refers to the principle that if a memory location is accessed, it is likely to be accessed again in the near future. i.e. the 80/20 rule model does not apply.
        
        ![[Screenshot_2023-12-01_at_2.28.14_PM.png]]
        
- Another potential reason is **hot memory does not fit in physical memory**
    - Imagine that the 80/20 rule still holds. But, the hot portion (pages that are frequently accessed) exceed the size of memory → the 20% is far larger than memory
        
        ![[Screenshot_2023-12-01_at_2.32.07_PM.png]]
        
- Each process fits individually, but too many for system
    
    ![[Screenshot_2023-12-01_at_2.32.22_PM.png]]
    

  

**→ Dealing with Thrashing**

_Approach 1:_ The working set model

- According to the 80/20 model, some portion of a programs address space will be heavily used and the remainder will not. This heavily used portion is called the **working set** of the process
- The set of pages current in primary memory are the **resident set**
- Primary memory (RAM) can be though of as. cache for secondary storage, i.e. it stores the recently used code and data
- **Thrashing viewed from caching perspective:**
    - given locality of reference (a.k.a given the tendency of a program to access a relatively small portion of its address space at any given time (20% part of the model), how much primary memory does the process need to avoid thrashing?
    - **In this approach, a practical approach to mitigating thrashing:** only run processes whose memory requirements can be satisfied. so, processes that can be accommodated in the available RAM will be allowed to run. Those with larger sizes than available RAM may be deferred to when there is more RAM available.

  

_Approach 2:_ Page Fault Frequency (PFF)

- In this, thrashing is **viewed as poor ratio of page faults to instruction execution**
- `PFF = Page faults / instructions executed`
    - If PFF rises above the **High** threshold (high rate of page faults relative to instructions), the process needs more memory. Not enough memory on the system? → swap out other processes pages
    - If PFF sinks below the **Low** threshold, memory can be taken away and given to other allocations

  

The following 2 graphs illustrate the above 2 approaches:

![[Screenshot_2023-12-01_at_3.59.40_PM.png]]

![[Screenshot_2023-12-01_at_3.59.27_PM.png]]

---

**→ Recall typical virtual address space**

![[Screenshot_2023-12-01_at_4.10.32_PM.png]]

The top of heap is called the **breakpoint**. Addresses between breakpoint and the stack are invalid → these regions will fault on access

The above is as we expected. Note that, as we’ve learned before, the **kernel** code is always at the top of virtual address space. Notice that the above has heap which grows upwards, and a stack grows down. Heap is used to allow us to allocate small pieces of memory that can be use by the allocation (`malloc` or `free`). Let’s look at the 2 main API’s used to implement the **heap.**

  

**→ Early VM System Calls**

- The OS keeps a **breakpoint**. There are 2 ways we can update the breakpoint
- Linux has system calls to allocate more virtual memory in the heap
    - `char *brk (const char *addr);`
        - This is the break system call → allows you to set and return new value of breakpoint
    - `char *sbrk (int incr);`
        - This system call increments value of the breakpoint and returns old value
- Can implement `malloc` in terms of `sbrk`
    - The OS gives `malloc` an initial amount of memory which it gives out in small slices, and `malloc` can request more when that initial allotment is insufficient
    - It is difficult to “give back” memory to the system

  

**→ Other memory objects can exist between the heap and the stack using** `**mmap**`

One of such objects are **memory-mapped** files. In the OS, between the heap and stack, we can have memory-mapped objects, namely **memory-mapped files**. These provide a way to map a file directly into virtual memory. This allows application, including multiple processes, to access the file’s contents as if it were in RAM → simplifies the I/O operations

- In such an instance, when a file is memory-mapped, the OS assigns a range of virtual addresses to correspond to the file’s contents.
- `void *mmap (void *addr, size_t len, int prot, int flags, int fd, off_t offset)`
    - Treat file as if it were memory → map the file specified by `fd` to the virtual address `addr` except that changes are backed up to the file. It can be shared with other processes.
    - If `addr` is `NULL`, then let the kernel choose the address
    - The virtual addresses you use in your program, after the `**mmap**` operation, are translated by the operating system into corresponding positions inside the file, not directly to physical addresses in RAM. This abstraction allows you to work with files as if they were in-memory data structures.
- `prot` — specifies the protection of region. It can have:
    - `PROT_EXEC, PROT_READ, PROT_WRITE, PROT_NONE (i.e. no access)`
- `flags`
    - `MAP_SHARED` - modifications seen by all processes sharing the mapping
    - `MAP_PRIVATE` - modifications are private to the mapping process and do not affect other processes sharing the same mapping
        - Imagine 2 processes both read and write to a file simultaneously. If they are both reading, flags are not that important. But, if a process writes, adding the `MAP_PRIVATE` means the other processes writing/reading to this mapped virtual address cannot see the changes. Adding `MAP_SDHARED` is the opposite
    - `MAP_ANON` → we include this flag when `fd` is -1 → i.e no file associated with this address range

  

Here are some more VM system calls:

- `int munmap(void *addr, size_t len)` → **remove** memory-mapped object from the address space
- `int mprotect(void *addr, size_t len, int prot)` → **change protection** on pages to `prot`
- `int msync(void *addr, size_t len, int flags)` → **flush** changes of `mmaped` file to backing store → when `mmaped` region is modified, changes are made in address space, and not reflected in the file itself.
- `int mincore(void *addr, size_t len, char *vec)` → When using `**mincore**` with a memory-mapped file, you can determine which pages of the mapped region are currently present in physical RAM and which pages have been swapped out to disk.

  

**→ Exposing Page Faults**

When there is a page fault, the program can be notified of this with `sigaction`. This function is used to specify **what action to take** for `SIGSEGV` _(the signal raised on invalid memory access)_ or any other signal. Consider the following code:

```Python
struct sigaction {
	union {.                         # system will have one of these signal handlers
		void (*sa_handler)(int);
		void (*sa_sigaction)(int, sigingo_t *, void *);
	};
	sigset_t sa_mask;
	int sa_flags;
};

int sigaction(int sig, const struct sigaction *act, struct sigaction *oact);
# sig - the signal for which the action is being specified
# act - pointer to `struct sigaction` that specifies the new action to be taken
# oact - if not null, save old sigaction to this
```

The code defines a structure (`**struct sigaction**`) representing the action to be taken when a signal is received and a function (`**sigaction**`) for setting up signal handlers.