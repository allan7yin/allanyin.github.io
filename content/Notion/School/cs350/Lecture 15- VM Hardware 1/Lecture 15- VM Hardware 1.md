### Virtual Memory - Introduction

**→ Want processes to co-exist**

Consider the following diagram:

![[Screenshot_2023-11-28_at_5.03.33_PM.png]]

Consider multiprogramming on physical memory:

- What happens if the allotted memory to **emacs** needs to expand?
- What if emacs needs more memory than is available on the machine?
- What if emacs has an error and writes to an invalid address → say 0x7100, which is not in its address space and in the **os’s** address space?
- When does **gcc** have to know it will run at address 0x4000
- What happens if **emacs** is not using its memory → should it just sit idly holding it? Or perhaps maybe we make it so some other program that needs the memory can use that space.

> The **Goal** is to manage memory in a way where multiple processes can share it. Let’s first see some of the issues that come with sharing memory:

  

**→ Issues in sharing physical memory**

There are 3 key issues with this:

1. **Protection**
    1. A bug in one process can corrupt memory in another
    2. So, must somehow prevent process $A$﻿ from trashing $B$﻿’s memory
    3. Also, must prevent process $A$﻿ from even observing $B$﻿’s memory → sensitive information from one process should not be accessible to other processes
2. **Transparency**
    1. Processes ideally should not be concerned with the physical location of their memory.
    2. However, in reality, certain processes may require specific physical memory characteristics. For example, processes often require large amounts of contiguous memory (for stack, large data structures, etc.)
3. **Resource Exhaustion**
    1. Programmers typically assume machine has “enough” memory
    2. Sum of sizes of all processes often greater than physical memory → Hence why we need some method to manage memory, to provide the illusion there is unlimited memory

  

We can avoid the above 3 issues through implementing **virtual memory.** Consider the following diagram:

![[Screenshot_2023-11-28_at_5.20.16_PM.png]]

  

- **Idea →** Give each process its own “virtual” address space
    - At run time, the **Memory Management Unit** relocates (i.e translates) each load and store address to an actual memory location. The app (in the above), or any user-level application in general, does not (and cannot) see physical memory directly
- The **MMU** also enforces protection
    - Prevent one app from messing with (i.e accessing) another process’s memory
- With this implementation, the **MMU** allows programs to see more memory than exists
    - Somehow relocate some memory accesses to secondary storage (disk)

  

**→ What are some of the advantages of Virtual Memory**

- First, we can re-locate a program while running → adjusting the programs memory addresses dynamically. For example, as a program runs, it may load additional modules or libraries into memory, requiring the OS to adjust the program’s memory addresses
    - We can move portions of it to different parts of physical memory → or even to disk via. **swapping →** moving memory to disk, and bringing it back when necessary
    - This works due to the **80/20 rule**. This rule suggests that 80% of our resources are going to be under utilized while 20% are going to be used. In the context of memory, this means for a large program, most of memory is not being actively used (idle). So, if we could swap that idle memory to disk, and leave only active parts in physical memory, and bring back idle memory only when the program reads/writes to those memory.
    - The **MMU** allows us to do these mappings and intercept these attempts to access idle memory → we’ll see later how this is implemented in software
- The challenge with virtual is that it introduces an extra layer → potentially slow because of this

  

Let’s briefly see some methods of implementations of VM:

- **Idea 1: Load-time Linking**
    - Link when process executed → not at compile time
    - Say program called C standard library, and calls `malloc` which it does not know location of at compile time. Dynamic linker finds it physical memory and sets it base address into `base` register, in the MMU. So, when the `malloc` symbol is resolved in assembly, that virtual address then adds the `base` register value to adjust. Next time print is executed, it’s virtual address is correctly used to translated to the corresponding physical address.
- **Idea 2: base + bound register**
    
    - The idea is, we have 2 specialized registers: **base** and **bound**
    - On each load/store:
        - Physical address = virtual address + **base**
        - Check 0 ≤ virtual address < bound, otherwise, trap to the kernel
    - This partially solves problems (faster load time → physical addresses are being directly converted using **base** when program’s virtual memory addresses are being used)
    - Still some downsides:
        - How to move this application if it needs to grow? → can only move other things out and find single chunk of memory that fits the entire application
            - We do this by changing **base** register
    - On context switches → OS must re-load **base** and **bound** registers
    
      
    

![[Screenshot_2023-11-28_at_6.25.45_PM.png]]

**So, quick summary:**

- Programs interact with virtual addresses
- Actual memory uses physical addresses
- VM Hardware is **Memory Management Unit (MMU)**
    
    - Usually part of the CPU
    - MMU’s registers that allow reloading of the **bound** and **base**, etc. → accessed with privileged instructions
    - Translates from virtual to physical addresses
    - Gives per-process view of memory called **address space**
    
      
    

**→ Base+bound trade-offs**

- **Advantages**
    - Inexpensive in terms of hardware: only need 2 registers
    - Inexpensive in terms of cycles: do add and compare in parallel (adding the base in parallel is cheap)
- **Disadvantages:**
    - Growing a process is expensive (have to relocate everything to a larger contiguous piece of memory) or impossible
    - No way to share code → what if 2 programs call a shared function? → e.g `C` library is used by every application. This should be easily shared if the code using it is read only.
- **One solution:**
    - To have multiple segments
    - Separate code, stack, and data segments and essentially, have multiple **base** and **bound** registers that can be used at the same time

  

This brings us to first design of VM that is still commonly used today: **Segmentation**

---

### Segmentation

Segmentation is a memory management technique where a program’s logical address space is divided into different segments. Each segment represents a different logical unit → code, data, stack, heap, etc.

- Segmentation allows processes to have many **base** and **bound** registers.
    - The address space is built from many segments (address space for both virtual memory and physical memory) → imagine if `C` library was one segment → shared

  

**→ Segmentation Mechanics**

![[Screenshot_2023-11-28_at_7.12.39_PM.png]]

If offset is invalid → fault. Otherwise, we proceed to verify the protection bit is valid (not writing to read only memory, above is read). If valid, add offset to base and find the physical address

The virtual address contains a segment number, and offset within that segment. This will be looked up in a table the processor has that contain the protection bits seeing whether its read, write, or execute (r in the above), the base address, and the length of that given segment. The **base** is used to add to the offset, while the **length** is used to compare to the offset to see if it is valid

- The **segment number** indexes the segment table
- The offset is the distance from the beginning of the segment to the specific memory location being addressed. Once the base address fo the segment is obtained from the table, the offset is added to it to determine the physical memory address of where the data is stored.
- In the segment table → **first one** is protection bit (read, write, execute).

  

If the offset is not valid (not smaller than **length/bound** in the segment table) or if the protection bits are violated, we will get a fault and allow the OS to decide what to do with the application. Each process has its own segment table with a set of segments. Let’s see an example of segmentation below:

![[Screenshot_2023-11-28_at_9.27.03_PM.png]]

For protection bits, first bit is for read, second bit is for write (rw)

- **Segment number** is expressed using 2 bits (0-3). So in the table labeled **virtual**, we have virtual addresses, such as `0x2000`. In this case, the first 2 bits (the 2) represent the segment. In the above diagram, the colours match the corresponding segment in the segment table. The next 3 digits (12 bits) represent the offset.
- The above also has a segment table. In it, we see the segment number, the base, bound, and protection bits. So, for example, we see that segment 0 is read only, whereas segments 1 and 2 are readable and writeable, and segment 3 is unused, so neither writeable or readable.
- Where are `0x0240`? `0x265c`? `0x3002`? `0x1600`?
    - `0x0240` → segment = 0, offset = 240 → 240 < `0x6ff` (4000) → PA is 4000 + 240 = 4240 **read only**
    - `0x265c` → segment 2, offset = 65c → 65c < `0xfff` (3000) → PA is 3000 + 65c = 365c **writeable and readable**
    - `0x3002` → segment 3, offset = 002 → segment 3 is not a valid segment number → **ERROR:** `**seg fault**`
    - `0x1600` → segment 1, offset = 600 → offset (600) exceeds bound (`0x4ff`) → **ERROR:** `**seg fault**`

  

**→ Pros and Cons of Segmentation Approach**

- **Advantages**
    - Allows multiple segments per process
    - Allows sharing! For example → Many segments point to the same `stdio` library routines
    - Don’t need entire process in memory
- **Disadvantages**
    - Requires additional translational hardware → could limit performance
    - Segments not completely transparent to program → this means the program needs to aware of or manage the segmentation details explicitly → so the way in which memory is organized into segments may have some visible effects or considerations that the programmer needs to be aware of.
    - This method still requires contiguous chunks of memory within physical memory, cannot be split. An `n` byte segment still needs `n` contiguous bytes of physical memory
    - This introduces the hazard of **fragmentation → a serious problem**
        - As we continually run and exit programs in memory, we will have gaps in between our programs called fragments. These are areas that we cant necessarily use until we move other programs around. Let’s take a close look below.

  

**→ Fragmentation**

![[Screenshot_2023-11-28_at_10.18.34_PM.png]]

By definition, this is the _**inability to use free memory because it is in too many small pieces**_. There are 2 types:

1. **Variable-sized pieces**: many small holes between pieces as memory gets freed and then partially filled by a request for a smaller pieces → called **external fragmentation**
    1. Consider the above diagram. The thin white bars are the fragmentation, they are chunks of memory that are too small to load another instance of something like emacs or gcc. But, notice the combined size of the fragments is around the same size, if not bigger, than the size of something like emacs or gcc. The only solution is the OS will stop applications, move them to compact memory, and then try to move the fragmented memory into a contiguous large chunk of memory.
2. **Fixed-size pieces:** no external holes, but force waste inside the pieces. This refers to the memory within a segment, that is not being used → internal waste

  

The logical way we address these issues is with **paging**. To do so, we make 2 main changes: **(1)** use fixed-size segments (called pages) and **(2)** we concatenate the segment number and the offset together.

---

### Paging

- **Idea:** Divide both virtual and physical memory into small fixed-size pieces (e.g. 4KB) called **pages**
- For each process, map its virtual pages to physical pages
    - Each process has its own separate mapping
- This allows OS to gain control on certain operations, such as:
    - Read-only pages trap to OS on write
    - Invalid pages trap to OS on read or write
    - OS can change mapping and resume application

  

**→ Paging Trade-offs**

![[Screenshot_2023-11-28_at_11.07.13_PM.png]]

Pages eliminate external fragmentation. This simplifies allocation, freeing and backing pages (with swap). Since most segments conceptually would take multiple pages, the average internal fragmentation is around 0.5 pages per “segment”. Let’s take a look at a simplified allocation:

  

**→ Simplified View Allocation**

![[Screenshot_2023-11-28_at_11.10.43_PM.png]]

  

Consider the above. We have 2 applications, assume they each run on a single segment → `gcc` and `emacs`. With paging, these segments (under the label `gcc` and `emacs`) are broken up into uniform size regions of memory → can then be mapped to arbitrary pages in physical memory.

- Note that, `gcc` in physical memory is not longer contiguous
- In the case of `emacs`, 2 of the pages are in physical memory, while one is in disk. If the program attempts to access any address in that 3rd page, the mapping should be marked invalid by OS → triggers a fault causing us to trap to OS → data is then read from the disk and loaded into physical memory.
    - Can store idle virtual pages on disk, and only bring them into memory when needed

  

**→ Paging Data Structure**

The paging data structures look **really similar to segmentation.** The key difference is the address of the segment and the offset, are always going to be concatenated together. It is very similar to the segmentation we saw above:

![[Screenshot_2023-11-29_at_12.41.28_AM.png]]

Some differences are:

- For segmentation: we had the segment number and offset → only part of the virtual address (the offset) was used to compute the physical address. In Paging, the entire virtual address is used for obtaining the physical address, so the virtual page number and offset are always concatenated together.
- Pages are fixed-size (e.g 4K bits)
    - The least siginificant 12 bits ($log_2(4k)$﻿) of address are **page offset**
    - Most significant bits are **page number**
- Each process has a **page table**
    - Maps **virtual page numbers (VPNs) to physical page numbers (PPNs)**
    - Also includes bits for protection, validity, etc.
- On memory access → translate VPN to PPN, which is obtained through the page table. Once we have the PPN, we concatenate the offset (hence why in the above, the PPN 1 is shifted 12 bits to the left to be able to concatenate with the offset)

  

**→ MMU Types**

The 2 main flavours of **memory management units are:**

1. **Hardware Managed:**
    1. Hardware reloads TLB with pages from page tables
        1. Recall TLB (Transition Look aside Buffer)is a hardware cache used in MMU. It caches page translations
    2. This uses some form of hardware, where page table data structures are specified by hardware
    3. Requires more complexity in hardware
2. **Software Managed:**
    1. Simpler hardware and asks software to reload pages
    2. Requires fast exception handling and optimized software.
    3. Enables more flexibility in the TLB (e.g variable page sizes)

  

In **virtual memory → each virtual address represents a byte of data (byte offset)**