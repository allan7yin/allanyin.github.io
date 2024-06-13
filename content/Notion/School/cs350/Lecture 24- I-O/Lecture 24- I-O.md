**→ I/O Bus**

- The CPU and its cores access physical memory (RAM) over **bus → communication pathway**
- Devices access memory over I/O bus with DMA (more later)
- Devices can appear to be region of memory (more later)
- A **crossbar** can connect any input to any output
- We can connect 2 different busses with a **bridge**

  

We often say RAM or physical memory, but what **is memory?**

- **SRAM:** very fast, used as cache for DRAM
- **DRAM:** just abbreviated to as RAM
- **VRAM:** video RAM → dual ported: can write while another component reads

What is I/O bus?

- High speed devices connected to processor

  

**→ Communicating with a Device**

- **Memory-mapped device registers (memory mapped I/O)**
    - Some physical addresses are mapped to device registers → memory **reads/writes** interact with device. So, devices and their control registers (below) are treated as if they were regions of the computer’s memory
- **Device memory:**
    - Device may have memory itself → OS writes directly to this memory **through I/O bus** instead of RAM
    - Some processors (x86) have special I/O instructions
- **Direct memory access (DMA)** → place instructions in RAM, and then using one of these special I/O instructions, we “poke” the device and ask it to perform an operation pointing it to some region of memory where operation request is
    - Device issue DMA request to read large amounts of data from memory

  

**→ Parallel Port**

To interact with I/O devices (such as monitor), on x86, there are 3 I/O control registers (called ports) → each allows us to control the bits on the physical port (imagine one of those spikey blue connectors that connect to monitor) itself. In particular, these 3 registers are:

1. **status:** return device’s current status → you **read** this register
2. **command:** issues command to device → you **write** to this register
3. **data:** data transferred to and from device with this

  

**→ Special IO instructions vs. Memory-mapped IO**

x86 has special IO instructions: `in` and `out` instructions used for input and output operations → allow CPU to communicate with devices by reading/writing to specific IO ports (the 3 registers above). This is not ideal since these instructions are **awkward** to work with. Memory-mapped IO can achieve the sam effect, by reading/writing particular addresses in memory that corresponding to the device. OS needs to map these physical addresses to virtual address → ensure values are **non-cacheable (remember this was a page table entry option)**. Consider the following:

```Python
volatile int32_t *device_control = (int32_t *) 0xc00c0100;
*device_control = 0x80; /* give command */ i
nt32_t status = *device_control; /* get back status */
```

- The `volatile` keyword in `C` tells the compiler that reads/writes to this address are to device memory → do not do fancy optimization, reads and writes must be in same order.
- At startup, the system assign physical addresses to devices

  

**→ DMA Buffers**

- Even with memory mapped IO, device interaction tends to be slow. We want to minimize the amount of interaction between CPU and device directly
- We use DMA buffers:
    - System sets up DMA buffers in memory → buffers act as temporary storage fro data that will be transferred to device and configures DMA controller about location of the buffers
        - Maintains a buffer descriptor list → each points to specific DMA buffer
    - Write this list of pointers to device. The device can then use DMA requests to access main memory without CPU
    - When a device needs to transfer data to or from memory, it initiates a DMA request. Controller facilitates data transfer from device to DMA buffer, and continues on the next buffer when one is done. While DMA controller is busy, CPU is free to execute other tasks.

Let’s see an example of this:

![[Screenshot_2023-12-10_at_3.46.33_PM.png]]

So, Link interface manages data to and from wire (like keyboard). This shows that when device communicates with system bus (all devices to interact with CPU), it is using DMA buffer. So, write to buffer, then the system bus will take those FIFO received, and copy them to main memory.

  

**→ Driver Architecture**

The device driver provides various commands or entry points to the kernel (like an API to the device). We need deriver to synchronize with device → driver needs to know when transmit buffers are free, or when request operation has error.

- One solution: **polling → device driver repeatedly checks status of device by querying at regular intervals**
    - When waiting to receive data, the driver continuously queries the device to check for the availability of incoming packets.
- **This has its drawbacks →** cannot use CPU for anything else when polling
- **Better →** ask device to interrupt processor when an event happens

---

In the last lecture, we look at calculations pertaining to disk → seek time, transfer time, rotational delay, etc.

- A **seek** consists of up to 4 phases:
    - **speedup -** accelerate arm to max speed or half way point
    - **coast -** at max speed (for long seeks)
    - **slowdown** - stops arm near destination
    - **settle -** adjusts head to actual desired track

  

**→ Cost Model for Disk IO**

The time it takes to move data to/from disk involves:

- **seek time** → depends on seek distance
    - Average seek time” quoted can be many things
    - E.g. time to seek 1/3 disk, 1/3 time to seek whole disk, etc
- **rotational latency** → time needed for desired sector to spin into place
- **transfer time** → time it takes to transfer data

> _**Request Service Time = Seek time + Rotational Latency + Transfer time**_

  

**→ Example 1:**

Parameters:

- Disk Capacity: $2^{32}$﻿ bytes.
- Number of Tracks: $2^{20}$﻿ tracks per surface.
- Number of Sectors per Track $2^8$﻿ sectors per track.
- Rotations per Minute (RPM): 10000 RPM
- Maximum Seek (time): 30 milliseconds

  

So, from this, we get:

- **Bytes in a track:** **$2^{32} / 2^{20} = 2^{12}$**﻿
- **Bytes in a sector:** **$2^{12} / 2^8 = 2^4$**﻿
- **Max rotational latency:** 10,000 RPM → so, 60 / 10,000 rotations/second = 0.006 seconds or **6 milliseconds**
- **Average seek time:** Max seek time / 3 → 30/3 = **10 milliseconds average seek time**
- **Rotational latency:** Max rotational Latency = 6/2 = 3 milliseconds average rotational delay
- **Cost to transfer 1 sector:** sector latency = ($6/2^8$﻿ = 0.0234 ms per sector)
- **Cost to read 10 consecutive sectors:**
    - Request Service Time = AveSeek + AveRotationalDelay + Transfer Time = 10 + 3 + 10*0.0234 = **13.234 ms**

  

Now, there can be multiple requests to disk at once, so placement and ordering of these requests is important to think about. We want to order requests to minimize seek times. Also want to achieve contiguous writes where possible. How?

**→ Scheduling: FCFS**

- Process disk requests in the order they come in
    - Easy to implement
    - Fair
    - **BUT →** cannot exploit request locality (we might be going back and forth, each time paying a large seek penalty)
        
        ![[Screenshot_2023-12-10_at_5.21.42_PM.png]]
        

  

**→ Shortest Positioning Time First**

- Always pick the request with shortest seek time
- **Advantages:**
    - exploits locality of disk to reduce seek time
- **Disadvantages:**
    - Starvation
    - Not always know which request will be fastest
        
        ![[Screenshot_2023-12-10_at_5.24.18_PM.png]]
        
        Another downside → can be unfair. 183 was second in queue, but runs last → being penalized for no reason
        

  

**→ “Elevator” scheduling (SCAN)**

- Like SPTF, but next seek is must be in the same direction
- Switch directions only if no further requests
- **Advantages:**
    - Takes advantage of locality
    - Bounds waiting time (unlike SPTF, where some requests may need to wait very long time)
- **Disadvantages:**
    - Still slightly unfair → cylinders in the middle get better service
    - Due to direction constraint, might miss some locality optimizations
        
        ![[Screenshot_2023-12-10_at_5.29.32_PM.png]]
        

  

**→ Flash Memory**

- Most computers today are using this
- Storing data with charges on memory cells → no mechanical parts to worry about → no seek time
- **limited durability →** It only has a limited number of overwrites possible → after this, the cells wear down and may be unable to store data