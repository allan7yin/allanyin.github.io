> These notes will begin from single-cycle data path to virtual memory and IO

DRAM and SRAM are types of RAM

- SRAM is more expensive, but faster
    - Implemented with D-Flipflops
- Dram is slower but cheaper
    

### Single-Cycle Data-path

Each operation has their **ALUop**, which will be a 2 bit value passed to the **ALU Control**, which then decides which operation to do in the ALU. Here are the **ALUop’s** for the various types of commands

![[Screenshot_2023-04-17_at_8.54.16_AM.png]]

For **D-Format & CB Format** instructions, don’t need to look at the Opcode, their ALUop is enough information to know what control input to pass to the ALU. For **R-Format**, however, we will need to look at the Opcode of the instruction to decide further:

- Specifically, look at bits **30, 29, 24** as those are different for **ADD, SUB, AND, ORR.** The opcodes for these instructions are on reference sheet.

  

So, the above tells us how to determine action of the ALU. Now, we look at what needs to come out of the control unit depending on what type of instruction we are executing. So, how can we determine what the instruction type is from the Opcode? In the opcode, we need only bits **30, 28, 27, 26, 25, 22** to know what kind of instruction. For **R-Format, bit 30** does not matter, and for **CBZ,** bit 22 does not matter (as its opcode is only 8 bits long instead of 11 bits).

![[Screenshot_2023-04-17_at_10.01.19_AM.png]]

Once we know the instruction type, we can set these values that will come out of the control unit. Practice filling in an empty one, to be better understand control lines in the single cycle data-path. **There should NEVER be a don’t care in MemRead, as if its not multiple of 4, will crash.**

  

We need to be able to determine the performance of single-cycle data-paths. Consider the following conditions:

1. Access to memory units take 200 ps
2. ALU takes 200ps
3. Register Files takes 100ps
4. No delay on other units
    1. Notably, this means control unit computes control line values in 0ps, so no delay
    2. We needed, we access 2 registers of the register file at the same time. If there is control unit needs more than 0ps, then there will be a delay in retrieving register 2, as we need to wait to know which bits (20-16 or 4-0) to check.

  

**Example 1) R-Format Instruction**

So, we would have: ==**200ps (Instruction memory) + 100ps (read from register file) + 200ps (ALU) + 100ps (write back to register file) = 600ps**==

**Example 2) LDUR Instruction**

So, we would have: ==**200ps (Instruction memory) + 100ps (read from register file) + 200ps (ALU) + 200ps (Read from memory) + 100ps (write back to register file) = 800ps**==

- For **STUR,** would be 700ps as would not need to write back to the register files in the end

**Example 3) CB-Format Instruction**

So, we would have: ==**200ps (Instruction memory) + 100ps (read from register file) + 200 (ALU) = 500ps**==

- We are checking if 0 (or not 0), so we’ll need to go through ALU (maybe just for pass B). If zero bit is asserted (meaning it is 0), we’ll go and change PC. Otherwise, PC+=4.

  

Another type of question you need to be able to do, is to modify the data-path.

**Example 1: Branch**

![[IMG_E0E289A89521-1.jpeg]]

![[Screenshot_2023-04-17_at_10.42.21_AM.png]]

So, the left is before, and the right is after. We can add the control line **Uncondbranch** which is set to 1 if the instruction is a pure branch instruction. We know that there are 26 bits used as offset for branch instructions. Since all 32 bits of the instruction are passed to sign extend, we can just say: **“suppose we add new hardware to the sign-extend unit, where it can sign-extend the 26 bits to 64 bits”.** Then, we add another or gate after the and grate near the top adder. So, as long as we have **Uncondbranch** set to true, we will take the offset’s value into pc.

  

**Example 2: BREL (Branch Relative Instruction)**

This is an **I-Format** instruction, it branches based on a register: **BREL Rn**, and sets PC = PC + 4*Rn, where Rn is in bits 9-5.

- To solve this, consider doing the following:
- We add a new control line, all it **BREL**, where it is set to 1 if we have a BREL instruction
- Then, we take the read data 1 and result from sign extend, and put them into a MUX with **BREL** as the select signal. 1 means choose read data 1, otherwise, sign-extend. The result of this MUX is then passed to shift left 2. It also goes into the or gate, to choose branching instead of PC += 4

  

The single-cycle executes a single instruction in a clock cycle.

**⇒ This wraps up single cycle data path - redo assignment 3 for practice**

  

### Pipelined Datapath

Pipelined data-path allows us to be executing multiple instructions at the same time. This ends up being a situation where, after the initial warm up stage, we are executing 5 instructions in a clock cycle, and still finishing one instruction per clock cycle. In this kind of implementation, each instruction will use 5 clock cycles. The 5 stages are shown below in the diagram:

![[Screenshot_2023-04-17_at_2.27.33_PM.png]]

They are:

1. **Instruction Fetch**
    1. Fetch instruction from instruction cache or memory
2. **Instruction Decode**
    1. Read registers and decode the instruction
3. **Execution**
    1. Execute the operation, or calculate a memory address
4. **Memory**
    1. Access an operand in data memory
5. **Write Back**
    
    1. Write the result back into register file
    
      
    

We are now able to continuously begin, and finish, tasks instructions in the pipeline. However, with this approach, certain hazards arise:

1. **Structural → Hardware cannot support this. We will not look at this too much as they are much too difficult to fix as these usually required a major design change**
2. **Data → Waiting for information that you don’t have**
3. **Control → Started instruction, but not the right one (we need to branch, but we’ve already begun the next instruction)**

  

**⇒ Data Hazard**

This occurs when we have something like the following:

![[Screenshot_2023-04-17_at_3.11.19_PM.png]]

To handle data hazards between **R-Format instructions like this,** we use forwarding. Now, there are other forms of data hazard. This one is also called **Load-Use Hazard.** Load-use hazards occur in situations like this:

![[Screenshot_2023-04-17_at_3.51.51_PM.png]]

We cannot solve with forwarding. So, we resolve these with stalls. To do a stall, we pass in a noop, which is just an operation that does not accomplish anything. Something like:

![[Screenshot_2023-04-17_at_4.27.08_PM.png]]

Another way to deal with hazards is through code re-arrangement. That is, we will go in an re-arrange the ARM code, to make sure that there are no hazards.

  

**⇒ Control Hazard**

This occurs when the instruction that was fetched is not the one that is needed. This can occur when we are branching. Since the pipeline cannot know what the next instruction should be, it reads in branch instruction, and next line. Now, this could be correct, but if the branch instruction is valid, then we should not be reading the next line. But, we already have, so something must be done.

- There are 2 solutions we will look at: **Stalls** and **Predicts**
    - For stalls, this would need to be a “magical world”. Meaning, suppose the hardware is able to compute the branch target address and update the PC in the ID stage. Then, we could stall for one stage, determine where PC goes, and resume.
        - VERY VERY VERY unlikely.
    - For prediction, the pipeline tries to predict whether the branch will be taken. If it is not, everything proceeds as normal. If it is taken, it stalls.
    - Delayed decision rearranges the code to move an instruction not affect by the branch to hide the branch delay. The rearrangement is done by the assembler, hidden from the programmer.

  

**⇒ Pipeline Registers**

Pipeline registers are used to make the pipeline for efficient/faster.

![[Screenshot_2023-04-17_at_4.41.24_PM.png]]

![[Screenshot_2023-04-17_at_4.51.06_PM.png]]

  

Now, we often no longer show the control unit signals anymore, but they are still there. But, with the pipelined data-path, we need to re-evaluate how to obtain Reg2Loc signal. Since the control unit is in the ID stage, it is in parallel with the register file. So, we need to get this signal directly from the opcode, which turns out, is always the **28th bit.**

  

Also, in diagram above to the right, notice that we extended the capacity of some pipeline registers. This is so that they can hold the control line values.

![[Screenshot_2023-04-17_at_4.55.11_PM.png]]

![[Screenshot_2023-04-18_at_10.38.55_PM.png]]

This means in the instruction set above, when **SUB X2, X1, X3** reaches WB and the **ADD X14, X2, X2** stage reaches ID, the value is first written back to the register file in first half of the clock cycle, and then read from in the second half, making it so there is no data hazard here.

![[Screenshot_2023-04-17_at_5.23.28_PM.png]]

![[Screenshot_2023-04-17_at_5.23.37_PM.png]]

⇒ Forwarding unit can give out signals 00, 01, or 10.

So, with those forwarding conditions, we resolve all data hazards relating to **R-Format Instructions.** However, this is not the case for D-Format instructions. For load-use hazards (when we load into a register in one instruction, then in next, want to use that instruction) can only be resolved through a stall.

  

If a stall is needed, all control lines are set to 0, and the instruction that is missing needed information waits to resume until the next clock cycle. After stalling for one cc, we can then use forwarding unit to resolve.

![[Screenshot_2023-04-17_at_6.48.28_PM.png]]

If we have detected that we need to stall, we will need to:

- Prevent PC and IF/ID registers from changing
- This stalls the instruction in IF and ID stages
- Must create bubble in EX stage
- Set all nine control signals to 0 in EX, MEM, WB control fields of ID/EX register does this

![[Screenshot_2023-04-17_at_7.00.24_PM.png]]

![[Screenshot_2023-04-17_at_7.00.35_PM.png]]

Now, from line 100 to 104, we have a load-use hazard. So, we set all control signals 0, and stall for one cc. However, as line 104 has already been read, the next line has already been read in, so now instead of running 104, we run 108. Also, our IF/ID register has now lost its old value.

- To prevent this, we give the **hazard detection unit** the ability to freeze the pc and IF/ID register for one cc

  

**⇒ Control Hazards / Branch Hazard**

The decision to branch does not happen until the MEM stage of CBZ. However, by the time we have already decided that we want to branch somewhere, an issue arises. The lines of code directly after the branch instruction have already been read into the pipeline, but now that we want to branch, they have been rendered useless and now present a hazard.

![[Screenshot_2023-04-18_at_5.26.46_PM.png]]

- If the branch is taken, the instructions in the IF, ID, and EX stage must be discarded, also called flushed.
- If you look at the pipelined data-path above, you can see that we could move the conditional branch execution earlier to the ID stage. In this case, the instruction in the IF stage needs to be discarded.
    - While this is true for unconditional branches, much harder to determine if it is conditional branch

  

What does a flush look like?

![[Screenshot_2023-04-18_at_5.30.38_PM.png]]

![[Screenshot_2023-04-18_at_5.30.47_PM.png]]

What we are doing here is similar to how we dealt with load-use hazards. Recall that we can only detect these data hazards in the execution stage. So, when the load instruction is in execution stage, and we detect a load-use hazard (so the next instruction, in ID, needs to use the load’s data), is then turned into a NOP, by setting all control bits to 0 going into ID/EX.

- The same concept applies for flushing. Only different is, in load-use, we freeze PC and save whats in IF/ID as we still want to run that instruction. But, in flushing, we don’t want to run those, so we don’t freeze PC.

  

Another solution is to decide branching in the ID stage. This way, where the above method flushes 3 instructions, now, we only need to flush 1. Here is a quick look at the hardware:

![[Screenshot_2023-04-18_at_7.28.04_PM.png]]

![[Screenshot_2023-04-18_at_7.48.53_PM.png]]

Now, we have moved the branching hardware into the ID stage. In the ID stage, we will still have the branch AND, but now, its in the ID stage instead of the MEM stage.

![[Screenshot_2023-04-18_at_8.05.56_PM.png]]

Now the above examples of stalling will then require new forwarding hardware. These will need to be forwarded to the “=0” unit. We won’t show how this is done, just know that if we are moving the branching hardware into the ID stage, then new forwarding hardware is required.

![[Screenshot_2023-04-18_at_8.13.15_PM.png]]

![[Screenshot_2023-04-18_at_8.13.45_PM.png]]

Now, currently, we are always working under the assumption that branches are not taken (hence why we are loading in the next instruction after the conditional branch command). This does not work well sometimes. What if we are in a loop? Most times, the branch will be taken. So under the current assumptions, we end up having to flush out IF register fairly often. We can reduce this by assuming the branch will be taken. To do so, we need new hardware.

![[Screenshot_2023-04-18_at_8.38.27_PM.png]]

We assume that there is a **Branch Destination Address Table** that is stores pc’s as index, corresponding to a branch address. So, when we encounter a branch, we go to the register to retrieve where to branch to. This is how we assume branch taken. If it ends up not being taken, we can always flush, and go to the next line (as pc is stored in IF/ID which can then +=4). We won’t learn how this works, just know it exists.

  

Still, there are many situations that are exactly if-statements or loops. So, this approach is not exactly ideal either. Another thing we can consider is **one-bit dynamic branch branch prediction.**

![[Screenshot_2023-04-18_at_8.46.01_PM.png]]

How does this work? if the branch taken bit is 1, we will take the branch. If this is right, good. If this is wrong, we change the bit to 0 so that next time we encounter this branch instruction, we do not branch. We can even improve this to a **2-bit branch prediction**

  

**Two-bit branch prediction** will only change the prediction fi two consecutive predictions are wrong. The previous decisions are remembered using a 4-state state machine. However, this is a minor improvement over one-bit. One-bit is already a good improvement. From that to here, is a minor difference.

![[Screenshot_2023-04-18_at_9.26.13_PM.png]]

![[Screenshot_2023-04-18_at_9.22.54_PM.png]]

![[Screenshot_2023-04-18_at_9.23.35_PM.png]]

![[Screenshot_2023-04-18_at_9.23.43_PM.png]]

![[Screenshot_2023-04-18_at_9.24.12_PM.png]]

### Compiler Issues

The compiler issues we will consider are code rearrangement and loop unrolling.

**⇒ Code Rearrangement**

This is something the compiler does, behind the scenes. Consider the following 2 slides:

![[Screenshot_2023-04-19_at_10.34.05_AM.png]]

![[Screenshot_2023-04-19_at_10.34.15_AM.png]]

  

By the nature of a pipelined data-path, we can re-arrange certain code so that the overall functionality of the program does not change, but now, we have a reduced the number of hazards, improving performance.

  

**⇒ Loop Unrolling**

When we see loops, we know that a certain amount of code is going to be repeated (re-executed). Instead of actually having a loop, and branching to the beginning of the loop, we can consider simply replicating the “looped” code physically.

![[Screenshot_2023-04-19_at_10.43.37_AM.png]]

![[Screenshot_2023-04-19_at_10.43.50_AM.png]]

  

In the above, we fixed the load-use through code rearrangement a while ago, so don’t mind it. Now, before unrolling, we took 14cc, as we repeat the 5 instructions 2 times, but with one instance of stall at the branch hazard. But, if we unroll the loop, we reduce the need to even do the last 2 commands. So, we have the initial setup (2) and the 3 loop lines repeated 2 times, giving us a total of 8cc. But of course, as you know how cache and virtual memory works, this approach will be use more space and memory. **For code rearrangement, there are some rules we want to follow:**

![[Screenshot_2023-04-19_at_10.50.25_AM.png]]

### Exceptions

Exceptions can be thrown at any given time. Something as simple as **ADD X1,X2, X1** could experience a “hardware malfunction”, resulting in a exception being thrown

- To handle exceptions, the processor saves the address of unfortunate instruction in the **ELR (Exception Link Register)**. Then, control is given to the OS at some specific address
- The OS will take the appropriate action, and can either: (1) terminate the program, or (2) Using the ELR, determine where to restart
- The OS needs to know what exception was thrown:
    - Vectored interrupts: A table, that tells you what exception maps to what address to give control to OS (not used by ARM/LEGV8)
    - ESR (Exception Syndrome Register)

  

**⇒ New hardware**

- ELR - Stores the 64 bit address where the exception was thrown
- ESR - Stores the 32-bit long cause for the exception. There will be a table of sorts, mapping exceptions to values. In CS251, we use ARM, which uses single-entry point for all exceptions. So, exception found? We always be reading in the same next line (exception handler)

![[Screenshot_2023-04-19_at_10.03.38_AM.png]]

![[Screenshot_2023-04-19_at_10.03.52_AM.png]]

We also want to have control signals (although we won’t show, just know about these), going from places like ALU (for overflow bit as control), IF (for unaligned memory access), ID (undefined instruction), MEM (unaligned memory access), so that those addresses of those instructions can be stored into ELR.

  

Consider the example below:

![[Screenshot_2023-04-19_at_11.13.53_AM.png]]

![[Screenshot_2023-04-19_at_11.14.00_AM.png]]

So, this means that the instructions in IF/ID now becomes no-op through the control line going there, then the instructions in EX and ID become no-ops, with the address of ADD instructions going into ELR. Then, the next cc, the location of the exception handler code begins to run. This is seen in the left of the PC mux, (imaging there is another MUX choosing between PC and the exception handler address).

![[Screenshot_2023-04-19_at_11.19.14_AM.png]]

![[Screenshot_2023-04-19_at_11.19.21_AM.png]]

The pipeline has up to five instructions active in any clock cycle, and each can generate exceptions. Prioritize the exceptions to determine which to service first. For example, earliest instruction is interrupted.

![[Screenshot_2023-04-19_at_11.19.53_AM.png]]

![[Screenshot_2023-04-19_at_11.19.59_AM.png]]

---

![[IMG_AB52E271EB55-1.jpeg]]

In the above, k is size of cache, ls is line-size, n is how many ways the cache is.

Giga is 2^30, tera is 2^40, mega is 2^20, kilo is 2^10

**⇒ If we do STUR instruction, then we need to set the dirty bit to 1**

**⇒ Instruction cache is 4-byte aligned (1 word), so one read of MEM → 64/4 = 16 words**