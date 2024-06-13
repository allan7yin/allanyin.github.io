**→** `**Inode**` **allocation**

How is a new `Inode` allocated?

- Each file (including directories) created requires a new `Inode`
- If it is a **new directory** → new CG from parent
    - Choose one maybe one with greater than average # free nodes
    - Choose CG with smallest # directories
- Within CG → `Inodes` allocated randomly (next free)
    - Would liked related nodes as close as possible
    - OK, because one CG doesn’t have that many `Inodes`
    - All `Inodes` in CG can be read and cached with small # of reads

  

**→ New Fragment Allocation**

When is a new fragment allocated?

- Occurs when a system must allocate space when user writes beyond the end of file
    - This happens when a file is being extended, and the existing allocated space is insufficient to accommodate the new data being written
- **Allow last block to be a fragment (1/2 or 1/4 of a block) to reduce internal fragmentation**
    - So if a file is a sequence of blocks, we allow the last one to be a **fragment**
    - If already a fragment, may contain space for write → done
    - If the last block of a file is a fragment and the new write operation cannot fit inside that existing fragment, the system may need to deallocate the current fragment and allocate a new block or fragment that can accommodate the entire write, prioritizing the new data.
- **If no appropriate-sized free fragments available → break a full block into fragments**
- Problem: slow for many small writes
    - Partial solution:
        - There is a **stat** data structure for each file, where the field `st_blksize` informs us the optimal file system block size
        - So we allocate in this size → then, things like `stdio` library can buffer this much data

  

**→ Block Allocation**

- Goal: **optimize for sequential access**
    - Use block in the same CG
    - If the CG is fully full, find other CG with quadratic hashing
        - if CG n is full → try $n + 1^2$﻿, $n + 2^2$﻿, $n + 3^2$﻿,…
    - Otherwise, search all CGs for some free space
- **Problem:** don’t want one file filling up whole CG → spread this out
    - Otherwise, other `Inodes` will not have room for their data, and it will be placed far away when we want locality
- **Solution:** break larger files over many CGs
    - We need relatively large extents in each CG so sequential access doesn’t require too many seeks
    - an "extent" refers to a contiguous block of storage space allocated for a file.
    - large enough so that extent transfer time > seek time

  

**→ Directories**

- Directories are a type of file → a field in the `Inode` identifies them as directories
- **Directory contents are broken into 512-byte chunks**
- Each one has `direct` structures with the fields:
    - 32-but **i-number**
    - 16-bit size of directory entry → so each directory can have $2^{16}$﻿ many entries
    - 8-bit file type (e.g directory)
    - 8-bit length of file name
    - **Essentially,** each one of these contains data/pointers to multiple directory entries, whether these be sub-directories or files. Each of these is a `direct` structure with its own corresponding i-number.
- Coalesce (add remaining entries to another chunk) when deleting
    - If first `direct` in chunk deleted, set i-number = 0 (i.e not used)
- Periodically compact directory chunks
    - but can never store directory entry across 2 different chunks
    - recall that only 512 byte sector writes are guaranteed to be atomic during a power failure

  

**→ Updating FFS for the 90s**

- Try to allocate logically contiguous blocks contiguously (up to 64kB).
- Delay writes until ready to write whole cluster (if possible).
- solution: cluster writes
    - FS delays writing a block back to get more blocks
    - Accumulates blocks into 64KB clusters (could also be one), written all at once
        - Basically, we write to a buffer, and once that buffer is full → then we write all at once
    - Keep track of which 64KB cluster is free and which is not using **cluster bit map**

  

**→ Crash Recovery: Fixing corruption**

- We must run FS check `fsck` utility after a crash
- File system’s summary info is usually inaccurate after a crash
    - scan FS to check
- System may have corrupt `inodes` → not a simple crash
    - bad block numbers, do a sanity check
    - cross-allocation → block used in more than one place
        - More than one file/directory thinks that data block belongs to them
- Fields in `Inodes` may be wrong → count # of directory entries to verify count (if no entries, but count ≠ 0, move to lost+found)
    - lost+found is directory for storing recovered files after file system check operation
- Directories may be bad:
    - All directories must be reachable and names must be unique

  

**→ Crash Recovery permeates FS Code**

- ensure that `fsck` can recover this when designing
- Permeating crash recovery into the file system code means that developers have implemented strategies and mechanisms at various levels of the file system's source code to handle and recover from crashes. This involves making sure that the file system can recover its state and maintain data consistency after an abrupt interruption, such as a power failure or hardware malfunction.
    - For example: You delete one file, then append to another file, then crash
    - The file system may reuse the block that was just cleared, and allocate it for the append operation
    - Risk of **cross-allocation:** both files think they own the block (deletion not sucessful)

  

**→ Crash Recovery Idea: Ordering Updates**

The OS must always update the file system components in a **consistent order**

- Write new `Inode` to disk **before** directory entry
- Remove directory name **before** deallocating node (don’t want name to exist but node to not)
- Write cleared `inode` to disk **before** updating CG free map

**Solution:** make metadata updates synchronous

- Doing one write at a time ensures consistent ordering → of course this hurts performance, but necessary for crash recovery
- **note**, the system cannot update buffers on the disk queue (a queue or list of pending disk operations) → blocks waiting to be written to disk
    - Say we write something then delete that same thing → while this can be nullified to 0 disk operations in soft updates since we can update the buffer in disk queue, here, we cannot. So, we will have to write toe disk, finish, then delete.
- Ordered updates enforce this strict order which hinders efficiency

  

**→ Crash Recovery: Performance vs consistency**

- Crash recoverability comes at a huge cost → makes some actions like `untar` (extracting files from archive) 10-20 times slower, all because you **might** lose power at any time
- While there are some hardware solutions, not the best ideas. The best solution is to implement advanced file system techniques to avoid crashing in the first place. Let’s take a look at what these are

---

**→ Soft Updates: First Attempt**

- **Goal:** want to avoid crashing after “bad” subset of writes
- Must follow 3 rules in ordering updates above:
    - Never write pointer before initializing the structure it points to
    - Never reuse a resource before nullifying all pointers to it
    - Never clear last pointer to live resource before setting new one
- Following these steps, the file system will be recoverable → can do so quickly, might leak free disk space
    - How do we achieve this? **Keep a list of what has happened on buffered blocks**
- **Example:** Create file $A$﻿.
    - Initially, block $X$﻿ contains an `inode`
    - Initially, block $Y$﻿ contains a directory data block
        - Directories are stored like files e.g. they have an `inode` which points to some data blocks
        - Those data blocks are the directory blocks, and store a list of files that are contained within the directory, along with their names, etc.
        - We have to create file $A$﻿ first before adding it to directory entry
    - Task: create file $A$﻿ in `inode` block $X$﻿ and the directory in block $Y$﻿
- We say $Y \rightarrow X$﻿, pronounced “Y depends on X”
    - which means $Y$﻿ **cannot be written before X** is written
    - **X** is called the **depndee,** and **Y** is **depender**
- Can delay both writes, so long as the order preserved
    - Say you create a second file **B** in blocks **X and Y**
    - You only have to write each out once for both file creations

  

So, this makes intuitive sense, to avoid crashing due to these mis-ordered writes (just like how referencing a deleted pointer’s name will crash a program), the above helps us to avoid crashing from bad writes. But, the above first attempt has one main issue: **cyclic dependencies**

- Suppose we create file **A** and unlink file **B →** both in same directory block and inode block
    - When you create a file, it is associated with one or more directory entries (links) that point to the file's data on the disk. Unlinking removes it as a directory entry. **BUT, this does not delete the files data from disk, and remains on disk as long as there are still existing links pointing to it.** So, when we unlink the file, we cannot delete remove that `Inode` until the pointer to it in the directory entry is cleared first
- **Cannot write directory until A’s** `**inode**` **is initialized by rule 1**, initialize struct then pointer
- **Cannot write** `**inode**` **block until B’s directory entry cleared by rule 2**, nullify pointer before reuse
- This is because we write entire `Inode` block and directory block to memory contiguously → so recall above in FSS, we write this to a 64KB cluster → more efficient this way

![[Screenshot_2023-12-08_at_3.44.56_PM.png]]

So in the above, you can see the cyclic dependency when we remove file B: we need to write to the directory block before the `inode` block, but at the same time, we also need to write to the `inode block` before the directory block → cannot progress. In addition to cyclic dependencies, there are other problems as well:

- Crashes might occur between ordered but related writes
- Enforcing this strict ordering is inefficient
- **Block aging →** block that always has dependency will never get written back
- The solution to this is **soft updates**:
    - write blocks in any order
    - **But,** keep track of dependencies
    - When writing a block, temporarily roll back any changes you can’t yet commit to disk
        - i.e you cannot write a block with any arrows pointing to dependees … but you can temporarily undo whatever change requires the arrow
        - Let’s consider the above example again, but with rollback

> **CONCLUSION:** so, with soft updates, we do not follow that strict ordering of writes, but the result of using rollbacks is that we imitate the behaviour of the strict oerding. This allows us to have fast crash recovery while avoiding the ineffeciencies related to strict ordering

  

  

![[Screenshot_2023-12-08_at_6.48.57_PM.png]]

  

![[Screenshot_2023-12-08_at_6.49.08_PM.png]]

These demonstrate this.

---

**Using soft updates means that** each update to a block in the list of updates that the file system keeps track of will now contain an old value, new value, and list of updates it depends on.