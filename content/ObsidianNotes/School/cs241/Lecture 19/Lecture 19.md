### Topic 15: Linkers

Recap:

- **Assemblers:**
    - **What:** need 2 passes to translate labels
    - **Why:** so that labels can be used before they are defined
- **Relocating Loader**
    - **What:** need to track and adjust labels that were used in a `.word` directive
    - **Why:** so that we can load our program anywhere in main memory (as every program thinks they start at 0x0, which is not the case)
- **Linker:**
    - Will learn in this lecture

So, what does the linker do? You may remember a similar concept in CS246, where we looked at the steps in compiling multiple c++ files. It is the same here. With linking, we are looking to use multiple files for some code.

  

**⇒ Why Link Object Code Files**

**Answer:** so that we can break up large programs into several smaller modules. We break up large programs for the same reasons we do so in high-level languages (maintainability, easy to make changes, debug, organized, etc.)

- **Procedural Abstraction:** programmers just need to know interface, what the procedures do and if registers are preserved, and not how the subroutine is implemented
    - errors are easier to track down
    - can be reused in many programs
    - different people/groups can be responsible for different parts
    - avoid duplication of effort (same print routine being created multiple times)
- Collect related subroutines together (into a library)

  

⇒ **How to Link: Attempt 1**

- What if we just simply concatenate everything into one file, and assemble that one single file? Bad for maintainability - a small change in one small file would result in redoing everything
- **Requirement 1: We need a tool that works with multiple MERL files as input**

  

**⇒ How to link: Attempt 2**

- What if we first assembled each file, and then concatenated afterwards? The problem with this lies with pc. Each program thinks it starts at address 0x0, so, all the files would start at the same position. This won’t be true when linking together multiple MERL files.
- So, if we concatenate multiple MERL files, the result is not a valid MERL file
- **Requirement 2: We need a tool that outputs the MERL format**
- **Requirement 3: We need a tool that works with labels (representing rub-routines) defined in one file and used in another (basically, something to support** `**.import**` **statements)**

  

**→ How to Link: The External Symbol Reference (ESR)**

- Create an assembler directive, `.import`, that tells the assembler that this symbol (i.e. label) occurs in another file
- The assembler does not translate this directive into an instruction. The directive provides information to the assembler
- When assembling, initially assign the value of 0 to this symbol, but make a note in the MERL file that this symbol is not yet defined
- If you ever find it, after linking is complete, than report an error

  

**ESR Format:**

- In the **Relocation and External Symbol Table** section of the MERL file, create an ESR entry X`
- There is only one ASCII character per word, to represent the chars in the symbol (here a label) in order to make it easy to implement. The format is:
    
    ![[Screenshot_2023-04-05_at_1.43.06_PM.png]]
    
- The first word is always `0x11`, which signifies that whatever follows is an ESR
- **CONCERN: What if multiple files use the same symbol?**
    
    ![[Screenshot_2023-04-05_at_1.44.34_PM.png]]
    

**→ The External Symbol Definition (ESD)**

- We want to differentiate between a symbol meant for local uses (within a file) and one meant for global use (external to the file).
- We use the `.export` directive to indicate that other files may use (i.e. refer to) this symbol
- A symbol may only be defined once, but may be referenced as many times.
- Using `.export` is like declaring a variable global
- The `.import .export` pair links the definition in one file to its reference in another
    
    ![[Screenshot_2023-04-05_at_1.48.25_PM.png]]
    

**→ ESD Format**

- In the **Relocation and External Symbol Table** section of MERL file, create an ESD entry
    
    ![[Screenshot_2023-04-05_at_1.50.37_PM.png]]
    
- This is largely the same as the ESR entry, except we use `0x5` to represent the beginning of an ESD entry

  

### Creating MERL Object Files

Recall that in assembly, we make 2 passes:

- **Pass 1:**
    - Record the size of the file
    - When a `.word <label>` instruction is encountered, record the location
- **Pass 2:**
    - Output the MERL header
    - Output the MIPS machine code
    - Output the Relocation and External Symbol Table (create a Relocation Entry for each relocatable address)

  

**Now, we want to introduce additional tasks to complete in the each of the passes:**

- **Additional Pass 1 Tasks:**
    - When an `.import <symbol>` directive is encountered, reach each symbol that needs importing and the locations where it is referenced
    - When an `.export <symbol>` directive is encountered, record each symbol tat needs exporting and the location where it is defined
- **Additional Pass 2 Tasks:**
    - When outputting the Relocation and External Symbol Table
        - For each symbol that is imported create an ESR entry for each location where it is referenced
        - Create an ESD entry for each symbol that is exported

  

### Linker Pseudocode

1. Concatenate the programs
2. Combine and adjust ESDs with new locations
3. Use new ESDs to update old ESRs and replace them by RELs (i.e the reference is no longer an external reference it is now a relocation entry)
4. Relocate address both in the body of the code and in the Relocation Table for RELs

  

**Key Task: Like loading, addresses need to be adjusted after concatenating. We look at the above steps more closely.**

**⇒ Step 1:** Concatenate the Programs

![[Screenshot_2023-04-05_at_2.27.31_PM.png]]

**⇒ Step 2:** Combine and Adjust ESDs

![[Screenshot_2023-04-05_at_2.43.20_PM.png]]

**⇒ Step 3:** Use new ESDs to update old ESRs

![[Screenshot_2023-04-05_at_3.11.12_PM.png]]

**⇒ Linking Example**

For this example, consider that we are linking together 2 files:

- `f1.merl` has 0x100 bytes of MIPS instructions
- `f2.merl` has 0x80 bytes of MIPS instructions
- The code from `f2.merl` will be added to the end of the code from `f1.merl`
- The resulting file will be called `f.merl`

![[Screenshot_2023-04-05_at_4.02.32_PM.png]]

![[Screenshot_2023-04-05_at_4.02.46_PM.png]]

  

![[Screenshot_2023-04-05_at_4.03.34_PM.png]]

![[Screenshot_2023-04-05_at_4.03.44_PM.png]]

Now, we introduce 2 more passes:

**⇒ Pass 3: Edits to the Code: Resolving ESRs**

In pass 3, the ESR on line 0x54 (which was originally in `f1.merl`) gets resolved. **i.e the label** `**pr**` **refers to location 0x160**

![[Screenshot_2023-04-05_at_4.09.19_PM.png]]

**⇒ Pass 4: Edits to the Code: Updating RELs**

![[Screenshot_2023-04-05_at_4.10.47_PM.png]]