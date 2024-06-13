These processes are not using a single resource all the time → Not always using CPU, it switches between CPU and I/O (e.g disk). So, there are bursts of computation, and bursts of I/O.

- To maximizer throughput → maximize CPU utilization and also maximize I/O device utilization
- How do we do this?
    - Overlap I/O & computation from multiple jobs. So while one job is using the CPU, have another be in an I/O burst
    - **Response time** is very important

  

For most programs, they have very short CPU bursts (lasts couple milliseconds) before going to do an I/O request.

**→ FCFS Convoy Effect**

- A long running job prevents many shorter jobs that are after it from running.
- Without preemption, CPU-bound jobs will hold the CPU until they exit or require more I/O, but I/O is rare for CPU-bound threads. This results in periods where **no I/O requests are issued and CPU is held**
- Result: **poor I/O device utilization** because I/O bound jobs wait for the CPU
- Large CPU-bound tasks holding all the other threads up. Every other thread is blocked on I/O and now needs CPU, but this one large process is hogging CPU time, so nothing is using I/O right now, hence poor utilization. We try to address this in **SJF Scheduling**

  

**→ SJF Scheduling**

- **Shortest-job first (SJF)** attempts to minimize turnaround time (i.e time it takes for each process to finish). The key idea is to _schedule the job whose next CPU burst is the shortest_
- **2 schemes:**
    - **Non-preemptive —** once the CPU is given to the process, it cannot be preempted until completes its CPU burst
    - **Preemptive — Shortest Remaining Time First** or SRTF: if a new process arrives with a CPU burst length less than remaining time of current executing process, preempt the current one and run the new one
- What does SJF optimize? → gives **minimum average waiting time** for a give set of processes

![[Screenshot_2023-12-09_at_7.55.23_AM.png]]

What are some limitations to this?

- SJF doesn’t always minimize the average turnaround time
    - It only minimizes waiting time, which in turn, minimizes the response time
- SJF can lead to unfairness or **starvation** of jobs with longer bursts → they never run, always some other processes waiting that would complete sooner, and they would be prioritizes
- In practice, can’t actually predict the future and know when the CPU burst length, but we can estimate it based off of past bursts

  

**→ RR Scheduling**

- **Round robin scheduling** addresses fairness and starvation → run threads in round robin fashion
    - Preempt the job after some time slice (also called **scheduling quantum**)
    - When preempted, move to back of FIFO queue
    - (most systems do some flavour of this)
- **Advantages:**
    - Fair allocation of CPU across jobs
    - Low average waiting time when job lengths vary
    - Good for responsiveness is small number of jobs and good when the size of jobs varies
- **Disadvantages:**
    - RR suffers when working with jobs of the same size. For example, if I have 2 jobs, and each one need 100 CPU time, then with a small quantum of 1, we would observe:
        
        ![[Screenshot_2023-12-09_at_8.17.52_AM.png]]
        
    - Context switches are not free, and carry an associated cost, and this matters when we’re switching so often. **How does this compare to FCFS?**
        - RR Average turnaround time → (199 + 200) /2 = 199.5
        - FCFS average turnaround time → (100 + 200) / 2 = 150
    - **For same-sized jobs, RR performs worse than FIFO**

  

**→ Costs of Context Switch**

This is important to consider when we look at scheduling algorithms

- **Direct cost:** kernel needs processor time to save and restore registers for system calls, and switching address spaces (e.g using different page table)
- **Indirect cost:** more cache misses in TLB, CPU’s RAM cache, etc.

When we use a preemptive scheduler (which most modern day OS do), we need to define a quantum on which we interrupt a program to run another one.

- **How to pick time quantum?**
    - Want this to be much larger than the cost of context switch (yeah I know the units don’t make sense here → supposed to be abstract)
    - Want the quantum to be larger than the **majority of bursts →** but not TOO large where every next thread will have a smaller burst, and we end up running a FCFS
- This is nice if we have some way of visualizing what the program is doing, and can determine an optimal quantum. if we cannot, we often just use FCFS or RR scheduling.

  

**→ Priority Scheduling**

Yet another method of scheduling. In this:

- Associate a numeric priority with each process
    - e.g smaller number means higher priority
- Give the CPU to the process with highest priority
    - Can be done preemptively or non-preemptively
- SJF is already a priority scheduling policy → shortest CPU burst time gets highest priority
- Recall that the issue with priority type scheduling, is that there is **starvation → low priority thread may never run as high priority thread is always ready to run**
    - So, how do we address this?
    - **Sol: Aging → increase a thread’s priority as it waits**
- This brings us to the first real-world scheduler we will consider

  

**→ Multilevel Feedback Queues (BSD)**

- In this, we have 128 priority levels, where we’ve grouped certain priority levels together, with a total of 332 such groups where each group has a run queue.
    - Kernel runs thread on highest-priority non-empty queue
    - Round-robins among processes on the same queue
- **Process priorities are dynamically computed**
    - Processes moved between queues to reflect priority changes
    - If a process gets higher priority than the currently running process → preempt and run it

  

Let’s look at how the BSD scheduler computes these priorities

**→ Process Priority**

Remember, the key idea of how the scheduler works is it looks at the highest priority queue that is non-empty and run the programs in that queue in RR fashion. In between all this, we are constantly re-computing priorities of all the programs in the system. In this, there are 2 parameters that matter:

- `p_nice` — user-settable weighted factor (user set priority for a process) → ranges from **-20 to 20 negative ones are higher priority**
- `p_estcpu` — per-process estimated CPU usage
    - Incremented whenever timer interrupt found process running. i.e it got preempted instead of blocked waiting for I/O
        - This effectively decreases the priority of the program
        - Increment by some constant (was not mentioned what this is)
    - Decayed every second while process runnable
        
        ![[Screenshot_2023-12-09_at_9.32.09_AM.png]]
        
    - Load factor adjusts how fast decay occurs based on length of run queue → **load** is the sampled average length of the run queue plus short term sleep queue over last minute
    - The higher the load, the less `p_estcpu` is reduced (the fractional different is always n/n+1 → decreases as n gets bigger)
- When the program is runnable, but not actually running, every second, we decay the **estimated CPU → what multiplying it by load does**
- With this, we use this `p_estcpu` value to determine which priority the process should be in → this value is `p_usrpri / 4`:
    
    ![[Screenshot_2023-12-09_at_9.50.04_AM.png]]
    
- This is clipped if over 127 → we only have 128 priority levels, just put this in the last priority then
- `p_estcpu` not updated while the thread is asleep → instead, `p_slptime` keeps count of sleep time. When the thread becomes runnable, we update the `p_estpu` by doing:
    
    ![[Screenshot_2023-12-09_at_11.22.05_AM.png]]
    

So, this is all under the assumption we are working with a single core system. What if we are in a multicore system? There are some associated issues we face:

- Scheduler must decide on more than which processes to run→ must decide with core to run process on
- Moving between cores has costs
- **Affinity Scheduling →** try to keep threads on same core. So, there are 2 choices:
    
    ![[Screenshot_2023-12-09_at_11.25.29_AM.png]]
    
    - We can have a single scheduler that assigns jobs to all cores, **or,** can have individual scheduling decision per core and deal with load imbalances at a second level (looking across all cores and load balancing)

  

**→ Thread Dependencies**

Another interesting aspect is thread dependencies. Consider the following example

- Say **H** is a thread at a high priority and **L** is at a low priority
    - L acquires lock I
    - **Scenario 1:** H tries to acquire the lock I, fails, spins. L never gets to run due to its low priority
    - **Scenario 2:** H tries to acquire the lock I, fails, blocks. M starts running which has medium priority. Again, L never gets to run.
    - This is an example of **priority inversion →** lower priority thread blocks higher priority thread from making progress
- We address this with scheduling decisions.

  

**→ Priority Donation**

We can adjust priorities to deal with lock dependencies between threads. Here are 3 examples:

- **Example 1: L low, M Medium, H High priority**
    - L holds lock
    - M waits on lock, L’s priority raised to L1 = max (M, L) = 4
    - Then H waits on the lock as well, L’s priority raised to max(L1, H) = 8
- **Example 2: Same LMH as above**
    - L holds l1 and M holds l2
    - M waits on l1, so L’s priority raised to L1 = max (M,L) = 4
    - H waits on l2, M’s priority goes to M1 = max(M,H) = 8. This cascades to the thread M has dependency on. So, L’s priority raised to max(M1, L1) = 8

**In summary:** In every case, the solution is to basically take the maximum of all priority levels that are waiting on a lock or resource, and donate that to the tasks that are currently holding that resource → once that resource has been released, this priority can be given back.

  

**→ Borrowed Virtual Time Scheduler**

This brings us here, another very common scheduler type. This is based upon the idea of virtual time. The goal of this scheduling algorithm is to ensure all processes get a certain percentage of the CPU. In certain cases, we want one process to run next when it isn’t technically supposed to run next (the one with smallest amount of effective virtual time should). This is useful for threads that need to run within a certain timeframe to maintain performance (like a youtube video) → these are called **soft real-time threads**

- “virtual time” → abstract measure of the amount of CPU time a process has consumed. Say a process takes 10 seconds to run since the scheduler takes it off and on. But in an uninterrupted environment → would only need 5 seconds → this is virtual time
- **Idea:** run the process with the lowest effective virtual time
    - Ai → actual virtual time consumed by process i
    - **effective virtual time** **$E_i = A_i - (warp_i $**﻿ **$? Wi$**﻿ **$:0)$**﻿
- Each process i has a share of the processor decided by weight $w_i$﻿ → get $\frac{w_i}{\sum_j w_j}$﻿
    - When i consumes $t$﻿ processor time, track it as: $A_i $﻿ += $t/w_i$﻿
- This could have lots of context switches → add in a limit, called **context switch allowance**
    - Process runs for $C / w_i$﻿ before preemption
    - We only switch processes when $E_j \le E_i - C/w_i$﻿ is switching from i to j → $C$﻿ is wall-clock time (real life time)

  

So this above description only considers threads that are running. What about threads that are asleep? The virtual time of process that slept will be far behind the time of other processes in the system that continued to execute. This means when such a thread wakes up, it will starve everyone from execution (since its virtual time is so small). To avoid this, the BVT bounds how far behind a process can be with the **SVT (Scheduler Virtual Time) → global minimum of virtual time for currently runnable threads**

- We monitor currently runnable threads and take their virtual minimum time and use it as SVT
- When waking thread i from **voluntary** sleep: → $A_i = max(A_i, SVT)$﻿
- If involuntary, we want to wait this “catch-up”