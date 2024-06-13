## Topic 17: Heap Management

Recall from A8, that we did code generation for `new` and `delete`. This was possible through the procedures which were provided to us: `init`, `new`, and `delete`. This lecture, we look more closely at heap management.

  

**→ The Challenge**

When we were working with procedure arguments, return values, and local variables, these could all be elegantly handled with the system stack. This was also because stack frames for nested procedure calls and returns, followed a **Last in First out (LIFO)** pattern, which is suitable for stack. However, dealing with `new` and `delete` is a much more problematic issue. Why?

- These can be called in an unpredictable manner
- They may be called within if statements.
- They don’t necessarily follow a pattern like Last In First Out.
- E.g. if new was called in the order: `new a`; `new b`; `new c`; `a`, `b` and `c` could be deleted in any order

  

In addition, local variables disappear once the function they are declared them is popped of the stack, but dynamically allocated arrays can remain, even after the function has returned. Also, many dynamic structures can grow and shrink dynamically, as size is not known at compile time. For these kind of reasons, it is simply inefficient to store dynamically allocated memory in the system stack.

- **SOLUTION: Instead, another region of memory is reserved for dynamically allocated memory: heap**

  

Here is a quick diagram showcasing the typical layout in memory:

![[Screenshot_2023-04-07_at_2.27.37_PM.png]]

The left shows the typical layout:

- The **code, read-only data, and global data** have relatively low addresses, and they do not change size as the program runs (as they stay constant through the program)
- Stack has a relatively high values for addresses and grows towards the heap
- The size of stack is always changing, as functions are called (popped onto the stack), and returned (popped off of the stack)
- The heap is near global data and grows towards the stack
- The size of the heap can grow if a program requires more dynamic memory

There are **three** tasks to consider for memory management:

1. **Initialization**
    1. **System Stack:** done by OS
    2. **Heap:** `init`
2. **Allocation**
    1. **System Stack:** `push()`
    2. **Heap:** `new`
3. **Reclamation**
    1. **System Stack:** `pop()`
    2. **Heap:** `delete`

  

When we are writing procedures, we have always done so in a manner where we allocate and free stack frames in the system stack efficiently. On the other hand, how the heap is managed varies.

  

→ **Varieties of Heap Management**

- Memory can be **allocated** implicitly (the system will decide when and how much) or explicitly (the program tells how much and when, `new`)
- Memory can be **reclaimed** implicitly (the system will decide when and how much) or explicitly (the program tells how much and when, `delete`)
- Memory can be **allocated** in one size only, or many (variable) sizes. But, whats the difference between **explicit and implicit**

  

**→ Implicit vs. Explicit**

- Many languages (Java, Python, etc.) have **implicit/automatic memory management**
    - The program creates new objects and a procedure runs in the background that decides when to free up the memory for the object because it is no longer being used (**aka garbage collection**). This is why we don’t delete anything after we `new` objects in Java, they are cleaned up by the JVM afterwards
- On the other hand, languages like C++ have **explicit/manual memory management**
    - Need to call delete

  

There are many pros for **automatic memory management**, some of which include a avoiding or substantially reducing:

- **Dangling Pointer Errors:** using memory that has been freed because delete was been called too early.
    - Risks: if memory location is being used by another data structure, you are unintentionally modifying that data structure in an unpredictable way
- For example,

```C++
int *ia = NULL;
ia = new int[100];
delete [] ia;
.
.
.
.
ia[0] = 17; // DANGLING POINTER ERROR
```

- **Memory Leaks:** allocate memory but then have no pointers pointing to it.
    - Program slowly uses up more and more memory
    - Risks memory exhaustion (running out of memory)
    - Risk increases the longer the program runs
- Classic example is doing like:

```C++
int *ia = new int[100];
ia = NULL; we have lost pointer to the dynamically allocated array, cannot free this memory, hence, memory leak 
```

- **Deleting twice:** delete on the same memory location can risk crashing the program

  

However, thats not to say there are no cons to automatic memory management:

- Uses more resources (time to track memory usage)
- May have performance impact
- Possible stalls in program execution (i.e not good for some real time programming applications)

  

**Yet, for both manual and automatic memory management:**

- both still need to track which locations in main memory are:
    - being used
    - free
    - We will look at these for explicit memory management

  

**→ Explicit Memory Management**

Here is the basic approach:

- We carve out an **arena** from main memory (large contiguous area of memory) that gets allocated once and then is handed out in pieces called **blocks** using calls like `new` or `malloc`
    - We call this arena the **heap**
    - This arena provides an area of memory that `new` and `delete` procedures manage
    - `new` is easy to implement if there was no `delete` procedure.

  

**Approach 1: No Reclamation (i.e. memory is not re-used) Approach 2: Explicit Reclamation**

![[Screenshot_2023-04-07_at_6.58.08_PM.png]]

![[Screenshot_2023-04-07_at_6.58.58_PM.png]]

![[Screenshot_2023-04-07_at_7.36.57_PM.png]]

  

![[Screenshot_2023-04-07_at_7.51.30_PM.png]]

![[Screenshot_2023-04-07_at_7.51.44_PM.png]]

![[Screenshot_2023-04-07_at_7.52.22_PM.png]]

![[Screenshot_2023-04-07_at_7.51.56_PM.png]]

![[Screenshot_2023-04-07_at_7.52.08_PM.png]]