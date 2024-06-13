In this lecture, we begin to look into file systems, such as Finder in MacOS

### The Challenges of File System

- File systems (FS): traditionally the hardest part of OS
    - More research papers written about file systems than any other single topic
- Main tasks of file system:
    - **Don’t go away →** Need to boot it up
    - **Associate bytes with name (file name) →** named data objects
    - Associate names with each other (directories)
- File systems can be implemented in a variety of places. We’ll focus on hard disks and generalize later. We’ll start by focusing on files, directories, and some performance.

  

**→ Why disks are different**

- To begin, we want make ensure file systems persist state after a crash, and is permanent in comparison to something like RAM.
- It is also slow in to processors. Every year, the disk gets faster by just a few percent, while the processor speed doubles every 18 months.
- Disks are large (100-1000x larger than memory) → holds most of our data here
    - Constantly accessing data on the disk
- **Key challenge: How to organize a large collection of ad hoc information**

  

**→ Magnetic Disk vs Flash Memory**

Magnetic disk is the traditional physical disk that spins where flash memory are things like SSD. Let’s make a quick comparison between the 3 main types of memory we know: **disk, flash drive, RAM**

- The smallest (also applies for atomic) write for these 3 are:
    - **disk** and **flash** have a smallest write unit of a sector (not specific, just some smallest writeable unit)
    - **RAM** has a **byte** as smallest writeable atomic unit
- For everything else (random reads, sequential reads) → we have an order of speed as: **disk < flash < RAM**, as expected
- The cost associated with these per GB also follows the same ordering → the faster it is, the more expensive it is per gigabyte

---

(**personal note**) → Before moving forward, let’s just review what the disk looks like how memory is read:

![[Screenshot_2023-12-10_at_4.52.38_PM.png]]

- Imagine each layer of the disk, moving towards to the centre of the disk, is a co-centric circle. Each of these layers is called a **track**. To switch tracks, the needle (or read/write head) moves towards the centre or away from it, as indicated by the **red arrow**. **This is seek.**
- Now imagine each 3D layer of the disk → This is a **platter.** A disk consists of multiple circles stacked on top of each other, where each one is a platter.
- On each track, the disk can spin, moving the needle along that circle’s circumference. Some slice of the circumference is called a **sector → this is the same smallest addressable unit** on the hard disk, smallest unit for read/write
    - Usually, all sectors have the same size
- A set of contiguous sectors is a **block**

**→ Disk Review**

- Secondary storage reads and writes in units of **sectors**, _not bytes_
    - Read or write singular sector and possibly adjacent groups
    - Modify that byte
    - Write entire sector back to **secondary stage**
    - **Key:** if cached, no need to read sector in again, just reach from cache (reduce 1 I/O operation)
- **Sector = unit of atomicity**
    - Sector write done completely, even if the system crashes in the middle of operation. Hard drive saves up enough momentum and energy and momentum and energy to complete it without power.
- **Tradeoff:** Larger atomic units have to be created by the OS

  

**→ Some Useful Trends**

- **Hard disk bandwidth** (transfer speed) and **cost per bit improving exponentially**
- **Seek time** (measures the time it takes for a disk to position its read/write head to the desired track or cylinder where data is located → contributes to the overall latency or access time in retrieving data from a specific location on a disk.
- **Secondary Storage accesses are huge system bottleneck** and getting worse
    - The speed at which data can be read or written to disk is slower than components like RAM and CPU
    - **Key idea:** Bandwidth increase lets the system pre-fetch large chunks for about the same cost as small chunk
    - Trade bandwidth for latency if you can get lots of related stuff
        - So, we can transfer more, but at the cost of longer/slower speed

  

**→ Files: name bytes on disk**

Let’s build up an understanding on the set of abstractions that the file system provides.

- **File abstraction:**
    - User’s view: a file is a _named sequence of bytes_ or a named data object
    - So, some file like `foo.c`, that has our code, split across multiple sectors and they will be spread out over the disk (like the below)
        
        ![[Screenshot_2023-12-02_at_9.32.38_AM.png]]
        
    - FS’s view: a file is a collection of data blocks
    - **File system’s job: translate name and offset to data blocks**
- **File operations:**
    - Create a file, delete a file, read from a file, write to a file
- **Goal:**
    - want operations to have as few secondary access as possible (i.e group related things)
    - use minimal space overhead

  

The filesystem is simply a bunch of metadata for translating between a set of mappings. The concept is very similar to what a page table did:

![[Screenshot_2023-12-02_at_10.10.29_AM.png]]

**→ File systems vs Virtual memory**

- In both settings, the goal is **location transparency →** the ability to access data without knowing its location.
- In some ways, FS has easier job that VM:
    - The amount of CPU time we can spend on doing file system mappings and updating these mappings is fairly large to what the allotted time is for VM (most things need to be completed in a matter of cycles), hence no need for TLB
    - Page tables deal with sparse address spaces and random access, whereas files are often denser (0,…,filesize - 1)and **access is often sequential**
- In some ways, FS’s problem is harder:
    - Each layer of translation = potential disk access → each access is very slow
    - **_Space is a huge premium!_** While physical disk space might be abundant, the space within the system's cache or memory is at a premium. Caching is crucial for performance, but the cache is limited in size.
    - **File size range** is very extreme: many files will be < 10KB, while some files may be many GB large

  

**→ Some working intuitions**

Now, we continue to mention some general things about file systems:

- FS performance dominated by the number of FS accesses
- Transfer time is only a small part of the total access time:
    - Access time = **seek time** + **rotational delay** + transfer time.
        - 1 sector: 5ms + 4ms + 5$\mu$﻿s → 9ms
        - 50 sectors: 5ms + 4ms + 0.25ms = 9.25ms
        - **Can get 50x more data for only around 3% overhead!** This is really good. This requires the sectors to be physically grouped together, or located in close proximity to each other.
            - To get this outcome, we want the file system to be placing blocks like this
- All blocks in a file tend to be used together, sequentially
- All files in a directory tend to be used together
- All names in a directory tend to be used together

  

**→ Common addressing patterns**

There a few addressing patterns we’ll find in file systems. Below are 3:

1. **Sequential:**
    1. File data is processed in sequential order (common with editors, compilers, etc.)
    2. By far the most common one
2. **Random access:**
    1. Address any block in file directly without passing through predecessor blocks
    2. Common in things like databases
3. **Keyed access:**
    1. Search for blocks with particular value → usually not provided directly by OS

  

This brings us to the main part of file systems, namely, how are they layed out.

**→ Problem: How to track file’s data?**

- **File system management**
    - Need to keep track of where file contents are in secondary storage
    - Must be able to use this to map byte offset to data block
    - **The structure tracking a file’s sectors is called an index node or an** `**inode**`
    - `Inode`s must also be stored within the file system
- **Some things to keep in mind:**
    - Most files are small
    - Much of disk is allocated to large files
    - Many of the I/O operations are made to large files
    - We want good sequential and good random access

  

**→ An** `**Inode**` is a data structure that stores information about a file or a directory → this metadata includes things like file permissions, ownership, size, pointers to blocks in disk, etc.

---

**→ File System Attempt 1: contiguous allocation → allocates files like segmented memory**

![[Screenshot_2023-12-02_at_1.07.13_PM.png]]

- The first attempt is, when creating file, make the user pre-specify its length and allocate all space at once (contiguous)
- The `Inode` contents are: location and size
    - In the above short example, **location (base) and size (len)**
- **Pros:** simple, fast access for both sequential and random access
- **Cons:** external fragmentation

  

**→ File System Attempt 2: Linked files**

![[Screenshot_2023-12-02_at_1.15.20_PM.png]]

- A linked list of data blocks
    - Keep a linked list of all free blocks (maintain a list of the white blocks above, for future allocations)
        - `**Inode**` contents: pointer to file’s first block
        - In each block, keep a pointer to the next block
- **Pros:** Easy dynamic growth and sequential access → no fragmentation
- **Cons:**
    - Non-contiguous blocks cause poor access times → poor non-sequential access times
    - Pointers take up room in the block

  

**→ File System Attempt 3: FAT (File Allocation Table)**

![[Screenshot_2023-12-02_at_1.20.38_PM.png]]

- Use linked files, **but links reside in fixed-size FAT** rather than in the blocks
- So we have a directory holding all the files, and the FAT holds the linked list
    - e.g. the direct holds `a: 6`, so, first block is 6. We look into the FAT at index 6, where it tells us the next block is 4. Looking at index 4, the next block is 3. Looking at index 3, there is no next block.
- This approach is the same as the above (attempt 2), but instead of storing the linked list pointers inside of the block, we house them in external data structures.
- **Quick discussion on FAT:**
    - The entry size is 16 bits → $2^{16} = 65,536$﻿ entries
    - Given a 512 byte block → FS is at most $2^{16} \times 2^9 = 2^{25} = 32 $﻿ MB
    - Can we increase the size of the file system?
        - One way is to increase block size:
            - Pros: can handle larger files
            - Cons: more **internal fragmentation →** files don’t need blocks that big
    - **Space overhead of FAT is trivial** (space overhead of FAT is space required by the FAT data structure itself)
    - **Reliability** (The main problems of FAT) → how protect against errors?
        - Create duplicate copies of FAT on disk
        - State duplication a very common theme in reliability
            - if one copy becomes corrupted → can use the duplicate one
        - **Bootstrapping:** where is root directory?
            - Fixed location on disk
                
                ![[Screenshot_2023-12-02_at_1.48.15_PM.png]]
                

  

**→ File System Attempt 4: Indexed Files**

![[Screenshot_2023-12-02_at_2.45.29_PM.png]]

- Each file has an array holding all of it’s block pointers
    - Similar to page table so it will have similar issues
    - Max file size fixed by array’s size
    - Allocate array to hold file’s block pointers during file creation
    - Allocate actual blocks on demand using a free list (linked list of free blocks)
- **Pros:** both sequential and random access is easy
- **Cons:** mapping table requires large chunk of contiguous space … same problem we were trying to solve initially in Attempt 1

> **Recall:  
>   
> **When we looked at page tables, we saw the problems with segmentation and introduced the idea of page tables. If the address space is very large, then a flat single-level page table becomes impractical due to its size. A hierarchical structure allows for a more compact representation of the page table by organizing it into multiple levels → hence why we added page directories.  
>   
> We take this idea into this example of file system implementation as well. We are faced with the situation, where the the index table of each file, can get fairly big, and requires contiguous memory. We can reduce this by taking inspiration from page table hierarchy.  

- So, we solve this the **same way as multilevel page tables**: small regions with index array, this array with another array, …
    - **Cons:** more FS accesses, which are expensive → each level of the index structure involves accessing a different part of the file system

  

**→ File System Attempt 5: Multi-level Indexed Files**

![[Screenshot_2023-12-02_at_3.06.32_PM.png]]

So here, the `Inode` is no longer an array to pointers of a fixed number of levels, instead, the `Inode` consists of a list of pointers. Here is how it works:

- In the beginning, for small files, each direct pointer in the array points to data blocks on the disk directly **(**==**purple**== **blocks above)**
- As the file grows to a medium-sized file, one direct pointer gets repurposed, and they now point to **indirect blocks** which just introduce another hierarchical layer (==**yellow**== **blocks in the above)**, where indirect blocks then point to data blocks on disk
- As the file grows large, a new direct pointers is repurposed. It now points to indirect blocks, which point to another set of indirect blocks, and this goes to the data blocks **(**==**green**== **blocks)**
- In this implementation the **first 12 pointers** are dedicated for small files, the **13th** one is for medium sized files, where we get another **128 blocks by reading the indirect block.** Very large files then use the **14th block**, where we get another 128*128 blocks.
- **Pros:** simple and easy to build, fast access to small files
    - Maximum file length fixed but large
- **Cons:**
    - In worst case, we have to access `Inode`, the double indirect, and the indirect block, before accessing a data block → (assuming `Inode` is cached, 2 I/O operations for access)

  

**→ More about** `**Inode**`**s**

- `Inodes` are stores in a fixed-size array
- In earlier implementations, the size of the array is fixed when the disk is initialized → cannot be changed. It was located at a fixed location in disk as well:
    
    ![[Screenshot_2023-12-02_at_3.40.35_PM.png]]
    
- Why is this a problem?
    - Imagine I want to read all files in a directory. Well, the directory also has an `Inode` and it will use it to find the data blocks (moving right, red arrow). For each file in the directory, it needs to go all the way back to the `Inode` array moving all the way left, and then go to the data block, all the way right. This is **very expensive**
- BSD splits out `Inode`s across the disk:
    
    ![[Screenshot_2023-12-02_at_3.42.31_PM.png]]
    
    - We want to co-locate the `Inodes` with the data blocks belonging to that file near each other to reduce seek overhead
- _The index of an_ `_Inode_` _in the_ `_Inode_` _array_ is called an **i-number**
- Internally, OS refers to files by their i-number
- When a file is opened, the `Inode` is brought into memory → written back when modified and file is closed

  

  

**NEXT UP: What is in a directory and why?**