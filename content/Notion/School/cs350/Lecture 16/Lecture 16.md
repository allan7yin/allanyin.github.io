We continue with VM hardware in this lecture. In particular, we will look at 2 examples of MMU’s: **intel x86** and **MIPS**, where one is hardware managed while the other is software managed

> **Remember:** Both the page directory and page table are stored in physical memory

### Intel x86: Hardware MMU

**→ x86 Paging**

- Page tables are controlled. through 2 main registers: `%cr0` and `%cr3`
    - Paging is enabled by bits in a control register `%cr0`
        - Only privileged OS code can manipulate control registers
    - Normally, pages are always 4KB in size
    - `%cr3` register points to the 4KB page directory → this is the main data structure that describes the page tables. This is just a additional layer of hierarchy to make the management of page tables more efficient → the combination of the page directory entry and the page table entry enables the translation of a virtual page number to a physical page number
    - **Page directory:** 1024 PDEs (page directory entries) → each contains **physical address** of a page table
    - **Page Table: 1024 PTEs** (page table entries)
        - Each contains physical address of virtual 4KB pages
        - Page table covers 4MB of Virtual Memory → Total size of page table is 4MB, where 4MB/4KB = 1024 pages, as expected

  

**→ x86 page translation**

![[Screenshot_2023-11-29_at_10.30.36_AM.png]]

  

The page tables used in earlier 32-bit processors come in 2 levels:

- **Page directory → 1024 entries**
- **Page table → 1024 entries**
    - Each entry corresponds to a 4KB page and each page table represents 4MB of virtual memory
- The top of the diagram has a linear address, or a virtual address.
    - The `%cr3` register points to the page directory
    - The first 10 bits are used to index the page directory → the indexed entry in the directory is a pointer to the base address of the corresponding page table
    - The next 10 bits are used to index the page table which. This entry points to the base address to the 4KB page that is the physical address of that virtual address
- The above is a **radix tree** design → 2 level tree where at each level, there is a fixed size → can represent $2^{20}$﻿ pages

  

The above may be slightly confusing, so we go over a simple 2-level paging example below:

![[Screenshot_2023-11-29_at_11.17.22_AM.png]]

![[Screenshot_2023-11-29_at_11.27.16_AM.png]]

The above example is using page tables rather than radix trees to keep it simple. The `V?` means “is this entry valid?”. If a level 1 entry is not valid (so if the page directory entry is not valid), there will be no level 2 table. The **address translation is the same** as a single-level paging, but the **lookup is different.**

The diagram on the right shows the process of converting the physical address `0x58B4`.

- First, in the above 2-level paging, we only have 4 entries in the page directory and 3 entries in the page tables → so we only need 2 bits to index these each. The first hex digit of the virtual address is 4 bits (5 → 0101). The first 2 bits `01` correspond to the page directory index, which is `0x1`, which corresponds to **table 2** and the entry is valid.
- Then, the next 2 bits are also `01` which is `0x1`. So, in **table 2**, we take the `0x1` index which is the PPN `0x12` and is valid.
- So, we then concatenate the offset to the PPN → `0x128B4`, where the offset is the same.

This kind of arrangement can be generalized to more than 2 levels…

  

**→ x86 Page Directory Entry**

![[Screenshot_2023-11-29_at_11.44.39_AM.png]]

![[Screenshot_2023-11-29_at_12.03.44_PM.png]]

- For the **page directory entry:**
    - **Present** tells us if the page table below exists
    - **Read-write bit** allows us to mark the entire region readable or writeable
    - **User/Supervisor** bit tells us whether its application or kernel memory
    - Some bits used for cache control and caching within the TLB
    - One of the import ones is **Page Size** bit:
        - Typically will be 0 → 4KB
        - If it is 1 → means the page directory entry points directly to a 4MB region of memory (4MB page)
    - Accessed bit is a flag that indicates whether the corresponding page in physical memory has been accessed (read or written to) during a specific period.
    - Dirty bit is turned on as long as one of the pages of the corresponding page table has dirty bit = 1, propagated up
- For the **page table entry:**
    
    - We have the same set of bits as the page directory entry
    - Also have dirty bit: means the pages need to be swapped to disk before they can be freed
    
      
    

**→ X86 Hardware Segmentation**

x86 is interesting to look at in terms of a hardware managed TLB as x86 also has segmentation.

- x86 architecture **also** supports segmentation (e.g registers for code, data, and stack segments)
    - It first uses segmentation to translate an address by adding its base and checking against its limit, into a linear address or virtual address.
    - It then takes that virtual address and maps it a physical address using page tables
- So why would we want both? → **we don’t lol**
- Generally, you don’t want both paging and segmentation: too much overhead, but there are some fringe cases when it is used

  

**→ Making Paging Fast**

- How do we make paging fast? In the previous x86, the page tables require up to 3 memory references per load/store
    - First lookup the page table address in page directory
    - Look up the physical page number in page table
    - Access the physical page (as described above)
- _**For speed, each core caches recently used translations**_
    - This is what the **TLB** (transition look-aside buffer)
    - Typically, it has 64 2KB entries, 4-way to fully associative, 95% hit rate
    - Each TLB entry maps a **VPN → PPN + protection information**
- On each memory reference:
    - Check TLB, if entry present, get the physical address fast
    - If not, walk the page tables, and then insert the translation into the TLB for next time (Must evict some entry, maybe with LRU)

  

**→ TLB Details**

- The TLB operates at the CPU speed → small, fast (if it was not small, it would not be fast)
- **Complication:** what do we do when we switch address space? (since each process has its own address space, so view of virtual memory)
    - Flush TLB on context switch (e.g this was done in the old x86, and is quite costly)
    - Tag each entry with associated process’s ID (e.g MIPS)
- In general, **OS must manually keep the TLB entries valid**. How?
    - In x86, the the `invlpg` instruction is used by the kernel to invalidate a page translation in the TLB
    - This instruction must be executed after making changes to a page table entry that may be in use. For example, if a page table entry is modified, such as setting the dirty bit to indicate a page has been written to, the `invlpg` instruction ensures that the corresponding TLB entry is invalidated
    - Otherwise, subsequent memory accesses might use outdated TLB entries, leading to incorrect translations
- In multi-core systems, where multiple cores share access to the same memory, managing the TLB becomes more complex. **TLB Shootdown** refers to the process of invalidating or updating TLB entries across multiple cores.
    - When a page table entry is changed on one core (i.e the virtual address is now being mapped to a new physical address), the TLB entries on other cores that may have cached the old mapping need to be invalidated or updated

  

**→ Where does the OS live?**

The OS itself is a program, and the kernel needs to be loaded in physical memory for the system to operate.

- Is it in it’s own address space?
    - We can’t do this on most hardware → they don’t support giving a separate address space for the OS
    - This would also make it harder to parse `syscall` arguments passed as pointers
- So, in almost all modern operating systems, the OS is in the same address space as processes
    - Use protection bits to prohibit user code from accessing, reading, or writing to kernel memory.
- Typically all kernel text (source code for the kernel), and most data, is **at the same virtual address in every address space → upper half of memory**. What does this mean? While a process has its own view of application memory, the region of memory that corresponds to the kernel memory is going to be shared across all processes.
    - On x86, must manually set up page tables for this. The OS specifically keeps a set of page tables for the OS itself → shared potentially across cores. The rest of the memory below will be used by the application.
- In other words, for each process's address space, a portion of that is reserved for the kernel's address space (0x8000 0000 or higher)

---

### MIPS: Software Managed MMU

MIPS has a very different design of the MMU than x86. What it basically does, is it exposes the TLB directly to the OS. All loads and stores will be mapped through the TLB, and if they don’t exist in the TLB, we’ll get a trap to the kernel.

- Hardware has a 64-entry TLB
    - Recall that when we looked at interrupts and exceptions, there is a specific exception vector that was used for handling TLB misses
    - Since the TLB is small (only 64 entries, compared to the thousands of entries in an x86 TLB), we expect the OS will receive traps more often
    - It will have to service, in a optimized way, these requests from the TLB
- Each TLB entry has the following fields:
    - Virtual page number, Pid, Page frame (a.k.a physical page number, in MIPS, we have virtual page numbers and physical page frames), plus some other flags
- Much of the kernel is unpaged. This means specific portions of the kernel, such as code, interrupt service routines, and essential data structures are always present in physical memory — They are not subject to paging
    - All of physical memory contiguously mapped in VM → although the unpaged region for the kernel is at a high VM address, it actually corresponds to the beginning of physical memory
- Whenever a page fault occurs, the user TLB fault handler runs, and it is **very efficient**
    - 2 hardware registers are reserved for fault handler

The main benefit of this approach is that the OS is free to choose its own page table format. Let’s briefly look at what virtual memory looks like in the MIPS architecture.

  

**→ MIPS Memory Layout**

![[Screenshot_2023-11-29_at_4.19.09_PM.png]]

Notice 0x8000 0000 or higher is saved for kernel address space

MIPS provides a set of segments:

- `**useg**` is pageable user memory, and represents the first 2 GB of address space → so a process can be up to 2GB in MIPS
- The top 2 GB is kernel memory → there are 3 segments. We won’t really delve into these segments
- **In other words, top 2 GB is kernel space, bottom 2 GB is user space**

  

**→ MIPS TLB**

In the TLB, there are 64 entries, with each entry being 64 bits in size. They are:

- **PID:** Process ID
    - This allows for whats known as a “tagged TLB”
    - In order to reduce cost of flushing and invalidating the TLB, we can have multiple process’ TLB’s loaded at the same time.
    - So, we only have 64 entries. Imagine if we’re switching frequently between 2 processes and using very little memory, some of the TLB entries of the other process could still be a valid
    - In this case, we can just tell the processor what PID we’re running under
    - So, the mapping can be compared against the current PID to know if it is valid for the current process.
- **N:** No cacheable bit
- **D:** Dirty bit → writeable, being set to (1) means page in physical memory has been modified (written to) since it was loaded into memory.
    - This needs to be written to memory again, for TLB entry → written to RAM
- **V:** Valid bit
- **G:** Global bit — ignores PID during lookups. This page mapping is global to all of memory → may be used for some kind of shared page that all processes have

![[Screenshot_2023-11-29_at_4.58.47_PM.png]]

With a software managed TLB → we can now have multiple page sizes (whereas is x86, we could only have pages of size 4KB or 4MB). Now, we have have any multiples of 4 from 4 kiB–16 MiB

- 4 kiB, 16 kiB, 64 kiB, 256 kiB, 1 MiB, 4 MiB, 16 MiB

  

**→ More on TLB PID and Global Bit**

- **PID** allows multiple processes to have translations in the TLB
    - Different processes have different translations for the same virtual address
    - We don’t need to flush the TLB on context switch by setting the process ID
    - i.e for a match to exist, both the VPN and PID must match
    - We only flush the TLB entries when reusing a PID
    - Current PID is stored in `c0_entryhi` (a co-processor register, MIPS co-processor 0)
        - `c0_entryhi` is the high bits of TLB entry (32-63). If you look above, that is where **PID** is located in the entry
- **Global bit** used for _pages shared across all address spaces_ in `kseg2` or `useg`
    - Ensures the TLB ignores the PID field
    - TLB flush usually does not flush this

  

So, what are the TLB instructions in MIPS? The TLB functionality is provided by the MIPS co-processor 0 (COP0). There are 4 **instructions:**

- `tlbwr`: TLB write a random slot
- `tlbwi`: TLB write a specific slot
- `tlbr`: TLB read a specific slot
- `tlbp`: Prove the slot containing an address

For each of these instructions, we must load the following registers:

- `c0_entryhi`: high bits of TLB entry
- `c0_entrylo`: low bits of TLB entry
- `c0_index`: TLB index

  

**→ Hardware Lookup Exceptions**

- **UTLB Miss:** Generated when accessing `useg` without matching TLB entry
- **TLB Miss:** Generated when accessing `kseg2` without matching entry
- **TLB Mod:** Generated when writing to read-only page

  

The **UTLB (User TLB) handler is separate from general exception handler:**

- UTLB misses are very frequent and require a hand optimized path
- 64 entry TLB with 4 KB pages covers 256 KB process → very small. When a process exceeds that size, it will trigger a user TLB miss, requiring OS to reload the TLB. This was done by hardware in x86 (remember the 3 memory references to reload the TLB). Here, the OS runs an exception handler to process the TLB miss.
- Require more entries (expensive hardware) or larger pages

  

**→ Hardware Lookup Algorithm**

This is the algorithm used by the MMU to perform quick and efficient lookups in the TLB.

> **TLB IS A HARDWARE PIECE IN MMU → MMU is part of CPU**

- If the most significant bit (MSB) is 1 (i.e. virtual address >= 0x8000 0000 so it is a kernel address) and in user mode → address error exception
- If no VPN match → TLB/UTLB miss exception. Translation is not in TLB
- If PID mismatches and global bit is not set → TLB/UTLB miss → translation is not in the TLB
- If valid bit not set → TLB/UTLB miss exception → the entry is stale (translation information stored in the TLB is no longer valid or accurate)
- Write to read-only page, and TLB has that page marked as read-only, a **TLB Mod exception** is generated (we were trying to **mod**ify a page that was read-only)
- If **N** is set directly

  

1. **Hardware-Managed MMU:**
    - **Functions Handled by Hardware:** In a hardware-managed MMU, many essential functions related to memory management, such as TLB (Translation Lookaside Buffer) management, page table walking, and address translation, are primarily implemented in dedicated hardware components.
    - **Efficiency and Speed:** Hardware-managed MMUs are designed for efficiency and speed. Dedicated circuits within the MMU handle tasks autonomously without significant intervention from the operating system, contributing to faster virtual-to-physical address translation.
    - **Less Flexibility for OS:** The operating system has less direct control over certain aspects of memory management, and the hardware enforces the rules and mechanisms predefined by its design.
2. **Software-Managed MMU:**
    - **Functions Controlled by Software:** In a software-managed MMU, the operating system takes a more active role in controlling and managing certain aspects of memory management. This includes functions like TLB management, page table management, and handling of certain exceptions like page faults.
    - **Flexibility and Customization:** Software-managed MMUs provide more flexibility and customization to the operating system. The OS can implement specific memory management policies, page replacement algorithms, and other strategies tailored to the needs of the system and applications.
    - **Adaptability to Dynamic Conditions:** The OS can dynamically adjust memory management strategies based on changing workloads, system conditions, or specific application requirements.