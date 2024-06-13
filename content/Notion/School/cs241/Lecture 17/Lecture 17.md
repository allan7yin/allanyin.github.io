## Topic 12: Loaders

**⇒ The Need for Relocation**

- So far, we have been assuming that each executable was created from an individual file. However, typically many files are combined into one large executable file.
- When creating individual files, we star at address 0x00
- Consider the subroutine print:
    
    ![[Screenshot_2023-03-29_at_1.49.27_PM.png]]
    
- What happens if we combine this with another file? Say, it is added after a main.asm file. The print label is now no longer at address 0x00, rather, it is at, say, 0x100. Any references to print need to be adjusted (i.e **relocated**) if the procedure is moved to a different location in memory.
    
    ![[Screenshot_2023-03-29_at_1.52.36_PM.png]]
    
- To understand this better, let’s first investigate a simpler example: a single file and a relocating loader

  

**⇒ What is Loading?**

We know how to convert an assembly language program into machine code (via the assembler, taking instructions and decoding into binary values). But, how is this machine code, this series of 32-bit binary instructions, actually run?

- To run, some other program must be responsible for copying it from **secondary stage (SSD)** into **main memory (also called RAM, primary storage, or primary memory)** and then starting to execute instructions in that program.
    - processors only execute code loaded in main memory
    - Must be loaded into RAM in order to be run
- The **Loader** is responsible for loading other programs into main memory and preparing them for execution

  

**⇒ Types of Loaders**

1. **Relocating Loader:** determines where in main memory the program will reside and adjusts any references to labels
2. **A more modern approach: The Linker**
    1. determines the location (in virtual memory) and then the loader loads the file into main memory

  

**⇒ A Simple Loader**

```C++
loader(P)
// P is the program to load and run, P = P[0], P[1], ...
for i = 0 to codeLength-1.     // copy P into memory starting at 0x0 
	MEM[i] = P[i]
$30 <- 0x010000000               // set the address of stack
jalr $0                          // start executing P
```

There is a bit of a problem with the above loader: **What if programs are not loaded into main memory at location 0x0**

**Solution:** Address need to be adjusted (i.e relocated) depending on where in main memory the program is loaded

  

**⇒ A Relocating Loader**

```C++
loader(P)
// P is the program to load and run, P = P[0], P[1], ...
// determine memory needed, n, and a location in main memory, a
n = codeLength + space for heap and stack 
a = AllocateRAM(n)                    // a is the starting address
for i = 0 to codeLength-1             // copy P into main memory starting at a
	MEM[a + i] = P[i]          
$30 <- a + n                          // set address of stack 
place a into $3                       // start executing P
jalr $3
```

Now, lets take a moment to digest the above relocating loader:

- We first determine the size of the program P (the codeLength)
- **Allocate some main memory** starting at, say a, for the code, stack, and possibly a heap
    - This is why we have `n = codeLength + space for heap and stack`
    - Then, we allocate that much space with `a = AllocateRAM(n)`
- Then, we copy the program from secondary stage (SSD or HDD) into main memory (RAM) starting at a (hence we we have MEM[a + i], where i starts at 0)
- Possibly set up the program, e.g. pass parameters to the program by placing them in registers or in P’s stack
- Load the program’s initial address a, into some register, say $3
- Now, can execute the program via jalr $3
- Possible do some work at the end, e.g. `mips.twoints` will print out all the resgiter values
    - In real life, the OS will also free up resources allocated to this process, e.g. the main memory

  

**⇒ Changing a Program’s Location**

**Key Problem:** If a program gets relocated in memory, it affects the values of certain labels. So, whenever `.word` refers to a location, if we are relocating, we must add a to the value of the location:

![[Screenshot_2023-03-29_at_3.03.59_PM.png]]

  

![[Screenshot_2023-03-29_at_3.04.08_PM.png]]

  

![[Screenshot_2023-03-29_at_3.05.00_PM.png]]

![[Screenshot_2023-03-29_at_3.05.18_PM.png]]

![[Screenshot_2023-03-29_at_3.05.10_PM.png]]

The 3 slides on the left and above, are a quick example of how in relocation, if `.word` references a location, it must be updated.

The hard part is finding these `.word` that are locations. Remember, machine code is just a sequence of bits, so, how do we know which words are addresses that must be adjusted (vs. constants or instructions which do not need to be adjusted)

- **Answer:** We don’t know without additional information
- **Approach:** We must augment the machine code with information about which words need adjusting if the code is relocated
- This enhancement of machine code with this additional information is called **object code**

  

⇒ **MERL**

**MERL** is a format for a program’s machine code that includes information about what words need to be adjusted if the program is loaded into a location other than 0x0

- **MERL: MIPS Executable Relocatable Linkable File**
- This is cs241’s own simplified format
- MERL has three parts:
    - A header
    - The MIPS machine code
    - The relocation information

  

**Part 1: The MERL Header**

The header consists of three words (12 bytes)

- **Cookie**
    - the value is 0x1000 0002
    - it identifies the type of the file
    - this can be interpreted as the MIPS instruction `beq $0, $0, 2` which would skip over the header if executed (skips the next two components below)
- **FileLength:** the length of the MERL file in bytes
- **CodeLength:** the length of the header plus the MIPS machine code (which is also the offset to the Relocation Table)

  

**Part 2: The Body - MIPS Program**

This is the program in MIPS machine code. It works correctly if the program is loaded into main memory location 0x0c (i.e. the location immediately following the header).

  

**Part 3: Relocation and External Symbol Table**

This contains relocation information.

- **Format:** the word 0x1 followed by the location of a word in the MERL file that needs to be adjusted if the file is relocated
- This entry is called a **REL or Relocation Entry**
- This part also contains external symbol definitions and external symbol references (to be discussed later)

Here is an example of a MERL:

![[Screenshot_2023-03-29_at_3.23.25_PM.png]]

![[Screenshot_2023-03-29_at_3.54.50_PM.png]]

From the figure above, we can see where the 3 components are in the MERL file. This now introduces some new Loader Pseudocode:

```C++
read in MERL header
a = AllocateRAM(codeLength)     // space for code + heap + stack
for i = 0 ... codeLength - 1    // copy into RAM 
	MEM[a + i] = instruction[i]
for each REL entry.               // relocate REL address
	MEM[a + location] += a
initialize $30 // stack pointer
place a into $3 // start executing code 
jalr $3
```

![[Screenshot_2023-03-29_at_4.50.28_PM.png]]

  

**⇒ Modifications to Create a MERL Assembler**

Pass 1:

- record the size of the file
- start counting addresses at 0x0c (rather than 0x0)
- whether you encounter a `.word <label>` instruction
    - record the location

  

pass 2:

- output the header
- output the MIPS machine code (already do this step, in the current assembler)
- output the relocation table

  

---

### Topic 13: Code Generation - Procedures

**⇒ Review: Prologs and Epilogs**

- In the **Prolog** for the main procedure **wain**
    - import print
    - inititialize constants **(store 4 in $4, and 1 in $11)**
    - push the return address **($31)** onto the stack
    - initialize **$29** and push a stack frame onto the stack
    - store wain’s args **($1 and $2)** in the stack frame
- Body of **wain**
    - initialize the local variables in stack frame
    - generate code for the body of the procedure
- Epilog of **wain**
    - pop the stack frame off of the stack
    - restore previous return address to **$31**

  

### **A8 P1: Multiple Procedures**

Now, for Assignment 7, we focused on generating code for only the return statement. Now, for this assignment, we will be looking at procedures, which including now looking at the types as well as re-implementing the symbol table.

- When we now allow procedures to call other procedures, we have additional requirements
- **Key Challenge:** How can we handle ==(1) situations where we are storing register values that will be needed later when those values will be overwritten by the callee?== ==(2) Updating the frame pointer and pushing a new stack frame?== ==(3) passing arguments?==
- **Key Ideal:** each procedure will have its own prolog and epilog , which makes sense, as its like a new “wain”  
    each time  
    
- wain’s prolog and epilog will handle
    - tasks that only need to be done once (initializing $4, $11, imports, heap)
    - tasks unique to it, e.g. copying the program’s arguments onto the stack

  

**Question:** What is handled in the prologs and epilogs of procedures **e(), r(), and wain()?**

```C++
int e(...) {...}          // the callee 
int r(...) {...e(...)...} // the caller 
```

  

**Handled in each procedure’s** prolog and epilog

- manage its stack frame and the frame pointer, $29
- save and restore our caller’s return address, $31
- save and restore the other registers
    
    ![[Screenshot_2023-04-01_at_6.15.12_PM.png]]
    

  

**⇒ Q1: Saving Register Values: Three Approaches**

So, who saves the registers?

c) hybrid (recommended approach)

- Caller saves some registers and callee saves others
    - Caller saves $31 (because its value is overwritten when the instruction `jalr` is executed)
    - Callee saves the registers whose values it will modify
- This is the approach we’ve been following so far

  

**⇒ Q2: Who Saves the Frame Pointer, $29**

- We will have the caller `r()` save $29 to the stack. Then, inside the called procedure `e()` the $29 value will be updated accordingly (next available stack location $30 - $4). This is the preferred approach as it is easier to implement. So, the caller will save $31 and $29, and call the procedure. Then, it will pop these values once returned from the procedure
    
    ![[Screenshot_2023-04-02_at_10.43.13_AM.png]]
    

  

What about passing arguments? When we worked with `wain()`, we only passed 2 arguments. We always saved the 2 arguments onto the stack along with local variables.

  

**⇒ Passing arbitrary number of arguments**

1. Caller loads the arguments (expr1, expr2, …)
    
    ![[Screenshot_2023-04-02_at_11.02.35_AM.png]]
    
2. Now for something like `procedure -> INT ID(params) {dcls statements RETURN expr;}` we know that we are processing the `params` in the caller. In the callee, we will need to do:
    
    ![[Screenshot_2023-04-02_at_11.03.53_AM.png]]
    
3. Issue is that the caller has already placed the arguments onto the stack. So, when we enter the callee, and first save registers and then local variables, we get a disconnect:
    
    ![[Screenshot_2023-04-02_at_11.05.01_AM.png]]
    
4. So, what can we do? **SOLUTION:** save registers after local variables. Then, we need to adjust the offsets inside the symbol table as they are no longer relative to $29:
    
    ![[Screenshot_2023-04-02_at_11.08.43_AM.png]]
    

  

**⇒ Namespace Collisions**

![[Screenshot_2023-04-02_at_11.09.15_AM.png]]

![[Screenshot_2023-04-02_at_11.09.26_AM.png]]