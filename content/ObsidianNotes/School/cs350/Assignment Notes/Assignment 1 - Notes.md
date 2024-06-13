In this assignment, lots of interesting and important knowledge was introduced.

**⇒ Here is a high-level overview of what is happening when we execute a .c file:**

  

When you compile a C source file into an executable, several steps are involved in the process, including compilation, assembly, and linking. Let's break down the process and understand what gets put into segments and sections:

### **Compilation Process:**

1. **Compilation (**`**.c**` **to** `**.o**`**):**
    - The C source code (`**.c**` file) is compiled by the compiler (e.g., GCC) into an object file (`**.o**`). During this step, the source code is translated into assembly code and then further into machine code (binary code).
    - Result: The compiled object file contains machine code instructions and data.

### **Linking Process:**

1. **Linking (**`**.o**` **to Executable):**
    - Multiple object files, including any necessary library files, are linked together by the linker (e.g., `**ld**`).
    - The linker combines the sections from different object files, resolves symbols, and generates the final executable.
    - Result: The linker creates an ELF executable file.

### **ELF Executable:**

1. **ELF Executable File:**
    - The ELF executable file contains several components, including sections and segments.
    - **Sections:**
        - The linker organizes the executable into sections such as `**.text**` (code), `**.data**` (initialized data), `**.bss**` (uninitialized data), `**.rodata**` (read-only data), and others.
        - Other sections may include the symbol table (`**.symtab**`), string table (`**.strtab**`), relocation information (`**.rel**` sections), and more.
    - **Segments:**
        - The linker uses the information in the sections to create loadable segments in the Program Header Table.
        - For example, the `**.text**` section becomes a loadable code segment, and the `**.data**` section becomes a loadable data segment.

### **Execution:**

1. **Runtime Execution:**
    - When you run the compiled executable, the loader reads the Program Header Table to determine how to load the segments into memory.
    - Loadable segments, such as the code and data segments, are loaded into the process's memory.
    - The program starts executing from the entry point specified in the ELF header.

  

Generating the ELF is what we saw in cs241, where we had multiple binary executables and were linking them together, resolving this like imports/exports, dependencies, symbol tables, etc. Now, we are looking at taking that ELF, and loading things into memory to execute those executables.

  

### Sections vs Segments

**Segments:**

- **Purpose:** Segments define how the program should be loaded into memory. They are crucial during the loading and execution of a program.
- **Location:** Information about segments is stored in the **program header table** of the ELF file.
- **Attributes:** Segments include attributes such as virtual address (`**p_vaddr**`), offset in the file (`**p_offset**`), size in memory (`**p_memsz**`), size in the file (`**p_filesz**`), and permissions (read, write, execute).
- **Types:** Loadable segments (`**PT_LOAD**`) are the primary type of segments that contain data to be loaded into memory. Other types of segments may include dynamic linking information (`**PT_DYNAMIC**`), note segments (`**PT_NOTE**`), and more.
- **Used by:** Segments are used by the loader to set up the program's memory layout during execution.

  

**Sections:**

- **Purpose:** Sections are primarily used during the linking process. They organize and group related pieces of data and code.
- **Location:** Information about sections is stored in the **section header table** of the ELF file.
- **Attributes:** Sections include attributes such as name, type, flags, address, offset in the file, size, and alignment.
- **Types:** Sections may include `**.text**` (code), `**.data**` (initialized data), `**.bss**` (uninitialized data), symbol table sections, relocation sections, and more.
- **Used by:** Sections are used by the linker to organize the code and data in the object files and to resolve symbols and relocations. They are not directly used by the loader during program execution.

  

---

  

Now, let’s dig deep into what spawn calls are, and how they work. Consider the example:

```Python
cat "a.txt" "b.txt"
```

For something like this, the shell receives the arguments `[cat, a.txt, b.txt]`. The shell then re- solves the full path of `cat` to `/bin/cat` and calls spawn, here called `OSSpawn`:

```Python
char *path = "/bin/cat";
char *args = { "cat", "a.txt", "b.txt" };
OSSpawn(path, args);
```

The OS then reads the strings from user to kernel space. Why? Commands that involve file operations, system calls, or complex operations typically come with arguments (parameters). In these cases, the relevant arguments often need to be copied from user space to kernel space for the reasons mentioned in lectures (security, validation, memory protection, etc.).

  

Then, The OS allocates a page into which it reads in all arguments as a sequence of strings. That is, the OS is setting aside a specific region of memory (a page) to store the arguments provided to a program or system call.