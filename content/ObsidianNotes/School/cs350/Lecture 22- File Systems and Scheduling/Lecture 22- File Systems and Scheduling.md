So, the soft updates are a mechanism to make the uses of `fsck` less expensive. It prevents the file system from crashing due to a bad subset of writes → so `fsck` does not need to check scenarios of, for example adding `inode` to directory before writing it to disk, etc. Does not need to check, and fix, crashes caused form a bad subset of writes → won’t happen with soft updates.

---

**→ Soft Updates**

The structure for each updated field or pointer includes: **old value, new value, list of updates this update depends on**. The system can write blocks in any order:

- must temporarily roll back (undo) updates with pending dependencies

Some dependencies are better handled by **postponing (rather than rolling-back)** in-memory updates. Let’s consider a simple example:

> Example: creating a zero-length file **A**
> 
> - **Depender:** directory entry for A
> - **Dependee:** bitmap — must mark `inode` allocated before directory entry written
>     - i.e.: _directory entry → bitmap, inode_
> - **Dependee:** `inode` — must initializer before bitmap is written to
> - **old value:** empty directory entry
> - **new value:** file name and `Inode` number

  

So, what are some operations requiring `Inodes`? Let’s take a look:

**→ Operations Requiring Soft Updates**

1. **Block allocation:** _block pointer → free block bitmap, data block_
    1. Write the data to the data block and update free block bitmap **before** the block pointer in either the `inode` or the `indirect block`
2. **Block de-allocation:** free block bitmap → block pointer
    1. Write cleared block pointer to disk **before** the free block bitmap
3. **Link addition:** _directory entry → bitmap, inode_
    1. Write to `inode` and free `inode` map (if new `inode`) to disk **before** directory entry
4. **Link removal:** _bitmap,_ `_inode_` _→ directory entry_
    1. Must clear directory entry in directory block before updating the free `inode` map and clearing the `inode` in the `inode` block

  

SO, what are some of the issues with **soft updates? Consider:**

- `fsync` is the UNIX function to flush (write) file changes to disk. This also flushes **directory entries, parent directories, etc.**
    - So earlier, when you saw last lecture, when we writer the `inode` block and directory block to disk, a couple of times → each time is via `fsync`
- `unmount` is the UNIX function to flush all changes to disk on shutdown
    - Some buffers (as we’ve seen before) must be **flushed multiple times**
- Deleting large directory trees frighteningly fast as you nullify directory first → so the clearing of the `inodes` can’t keep up
    - So, dependencies are being created faster than blocks written → we **cap the number of dependencies** allowed to prevent resource exhaustion
- Useless write backs

  

**→ Soft Updates fsck**

We now revisit this. Remember this was the system call to perform thorough file system check:

- **Split** `**fsck**` into foreground and background parts
- Foreground must be done before remounting FS:
    - make sure per-cylinder summary info makes sense
    - recompute free block/inode counts from bitmaps → very fast
- Background does traditional `fsck` operations:
    - It can be run in the background, allowing users to access the file system during this operation.
    - Do after mounting to regain free space
        - When a file system is mounted, the files and directories stored on the mounted device become accessible
- Difference from traditional FFS `fsck`
    - May have many, many `inodes` with non-zero link counts
    - Don’t stick them all in `lost+found` (remember for FFS, if no entries, but count ≠ 0, move to lost+found), may be too many of this instance with soft updates

  

**→ An alternative: Journaling**

- The biggest crash-recovery challenge is consistency → one operation such as creating a file, requires many multiple disk writes. If not all of occur in time, we have a problem
- **Most of these problematic writes are to metadata**
    - While error in write on file only affects that file, error to metadata affects **numerous** files
- To address this, we use a **write-ahead** log to **journal** metadata (techniques used to ensure data consistency and recoverability)
    - So, maintain a **“write-ahead” log** that records the intentions of file system changes before they are actually performed → this action is **journaling**
    - This is reserved on disk
    - After crash → re-play the log (applying the changes)
    - May re-do already committed change, but won’t miss anything.
- log is typically split into individual log entries. Each log entry represents a discrete event, operation, or transaction within the system → **chronological order**
    - **group multiple operations into one log entry → either everything in this is replayed, or none**
        - e.g clear directory entry, clear `inode`, …
- log is typically split into individual log entries. Each log entry represents a discrete event, operation, or transaction within the system
- **Advantages:**
    - safe to consider updates committed when written to log
- **Example:**
    - When deleting a directory tree → record all free blocks and changed directory entries in log
    - Write out these changes asynchronously
- **Challenge:** Must find **oldest relevant log entry →** it is redundant and slow to replay the **WHOLE THING**
- To solve this, we use checkpoints:
    
    - Once all records up to a certain log entry have been stably committed to disk, set a checkpoint here. When there is a crash before the next checkpoint, read starting from here.
    - Play until the end of the log
    
      
    

> **COME VISIT B+ TREES ON THE LAST DAY**

**→ Journaling vs. Soft updates**

Note, these are not mutually exclusive, and we can use both. I think for this course, we view them as 2 separate solutions though. Both journaling and soft updates are much better than FFS alone.

- **limitations of soft updates**
    - Very specific to FFS structure
    - Still need the slow `fsck` to reclaim space
- **limitations of journaling**
    - disk write required for every metadata operation (log) whereas create-then-delete might require no I/O with soft updates
        - Say you quickly append block to file then truncate the file. You will know the block pointer is not written because of the allocated dependency structure.
        - So both operations together require no disk I/O! (for soft updates)
        - On the contrary, both these actions would be written to log → disk access

  

**This brings us to the end of file systems. We now start looking at CPU scheduling.**

---

**→ CPU Scheduling**

Problem:

- Have K jobs ready to run, have N ≥ 1 cores ready to run. **We want to assign jobs to cores**

Consider the following diagram:

![[Screenshot_2023-12-08_at_9.15.57_PM.png]]

- Recall all the way back, where we talked about scheduling processes. Now, we look at how thats done. Scheduling decisions may take place when a process:
    
    - Switches from running to waiting state (sleep thread)
    - Switches from running to ready state (CPU just making a context switch, no other reason)
    - Switches from new/waiting to ready
    - Exits
    
      
    

**Why do we care about scheduling? Our goal is to:**

- Maximize **throughput →** complete as many processes per unit of time
- Minimize **turnaround time →** time for each process to complete
- Minimize **response time →** time from request to first response
    - These above are also affected by **CPU Utilization →** fraction of time CPU doing productive work, **waiting time →** time each process waits in ready queue

  

**→ Example: FCFS Scheduling → First come first serve**

This is the first scheduling algorithm we will look at. So, if we get tasks p1, p2, p3 in that order → just process them in that order

![[Screenshot_2023-12-08_at_10.26.22_PM.png]]

- Really simple to implement
- Throughput → 3 jobs / 30 seconds → 0.1jobs/sec
- Turnaround time: → average turnaround time: 24 + 27 + 30 / 3 → 27 seconds
- **We can do better** simply through a better order → process P2, P3, then P1

![[Screenshot_2023-12-08_at_10.28.23_PM.png]]

- **Throughput** is the same, but turnaround time: average → 3 + 6 + 30 / 3 = 13 < 27 by a lot!

So, scheduling algorithm can reduce TT.