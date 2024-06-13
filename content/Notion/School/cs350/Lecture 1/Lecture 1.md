# Introduction to Operating Systems

**An operating system (OS)** is system software that acts as an intermediary between computer hardware and software applications. It provides a set of essential services and functionalities that enable users and software programs to interact with and make use of the computer's hardware resources effectively. In essence, an operating system serves as a platform for running applications and managing the overall operation of a computer or computing device.

  

The following diagram:

![[Screenshot_2023-09-14_at_10.17.16_AM.png]]

  

So what is a **kernel?**

  

**⇒ Kernel:** A kernel is a critical component that serves as the core or central part of the OS. It is responsible for managing various low-level hardware resources and providing essential services to higher-level software applications. The kernel is a very **abstract** concept. Think of it this way, **the kernel is like an orchestra conductor**. In an orchestra, you have various musicians playing different instruments, and they need someone to coordinate and manage their actions to create beautiful music. Similarly, in a computer system, you have various hardware components (CPU, memory, storage, input/output devices), and they need something to coordinate and manage their activities to make the computer work smoothly — the kernel.

  

The OS provides a layer of **abstraction** and **safety**(e.g interfaces) when processes (e.g. file system for vim), need to interact with things like I/O and memory.

- **Provides Abstraction** i.e. (1) managing and (2) hiding the details of the hardware.  
    - The OS accesses the hardware through low level interfaces unavailable to applications.  
    
- **Provides Safety** — prevent one process or user from clobbering another or reading/writing into the memory of other data structures

  

While the first thing you may think of when I say _OS_ is something like **MacOS** or **Windows 11**, there are many different types of OS, that are used in various different settings.

- **Library/Primitive OS**
    - **Uses:** Think of things like IoT Sensor Software. IoT sensors are devices that collect data from the physical world, such as temperature, humidity, light, motion, and more, and transmit this data to a central system or the cloud for further processing and analysis. In short, this kind of **OS** is often used in things like smart refrigerator, smart thermometers/AC, etc.
    - Library of standard services (without protection, as there is no need)
    - These make simplifying assumptions:
        - systems will run one program at a time
        - no bad users or programs
    - **Problem: (1)** Poor utilization of hardware (e.g CPU idle waiting for hardware) **(2)** Poor utilization of user (must wait for each program to finish)
- **Multitasking OS**
    - Can run more than one process at the same time, the kind of OS used in our laptops
        - While one process blocks, run another process
    - **Problem:** What harm can an ill-behaved process do? **(1)** Go into an infinite loop and never relinquishing the processor. **(2)** Overwrite another process’s memory, making it fail.
    - A multitasking OS must provide mechanisms to address these problems, e.g.
        - **Preemption** – take processor away from looping process
        - **Memory protection** – protect processes’ memory from one another.
- **Multiuser OS**
    - Idea: With n users, the system is not n times slower, because user demand for processor is bursty (e.g. read, type, select a menu item).
        - Student server — multiple users logged in at the same time
    - What can go wrong?
        - Users can use too much CPU time, storage etc. ⇒ need policies
        - Total memory usage may be greater than machine has ⇒ must virtualize
        - Super-linear slowdown with increasing demand, called thrashing.

  

Of course, there are OS’s that are multiple (MacOS is multiuser and multi-tasking).

With the above problems, there needs to be the needed safety features in place. We now introduce some definitions:

  

**Pre-emption:** is the the ability to give and take away resources such as processor time.

**Interposition** or **mediation** — inserting and intermediary layer or component between 2 entities to control or monitor interactions between them**:**

- Place OS between application and the resources
- Track all the resources that an application allowed to use (e.g., in table).
- On each access, look in table to check that the access is legal.

**Privileged** and **unprivileged modes** in processors (CPU):

- User applications have unprivileged access (also called **user mode**).
- The OS has privileged access (also called **kernel mode**) — available for those who develop the OS (so like Apple and Microsoft OS developers, etc.)
- Protection operations can only be done in privileged mode.

  

Consider a diagram similar to the above:

![[Screenshot_2023-09-14_at_11.01.39_AM.png]]

Most software runs in the **user space** such as vim, gcc, and every application I interact with on my laptop. Notice that the above, there is something called **device driver. Device drivers** in the OS Kernel are software components that serve as intermediaries between the kernel and hardware devices. They enable the OS to communicate with and control various hardware peripherals, such as disk drives, network adapters, graphics cards, USB devices, and more. This is the component that is specific to different hardware manufacturers (e.g HP, DELL, Apple, etc.)

  

We can now provide a very brief introduction into system calls. We will learn more about these latter on.

  

**⇒ System Calls**

- Applications can invoke the kernel through **system calls (an interface which is supplied by the kernel)**
    - Recall that, in order to perform various tasks or access system resources, user-level programs need to interact with the kernel by invoking its functions through a controller interface. This is what is meant by **invoking the kernel**
- These are special instructions (similar to a function call) which transfers control to kernel which in turn dispatches one of hundreds of system call handlers.

  

The above process is often called **trap** to kernel. It describes the process by which a user-level program transitions from user mode to kernel mode to request services or operations from the operating system kernel. So what is the goal?

- We want to execute functions that the app cannot do in unprivileged mode.

In fact, higher-level functions that an application programmer would use are built on this **system call interface**. E.g.

- `printf`
- `scanf`
- `gets`

  

Here is a quick diagram that demonstrates the basics of what this means:

![[Screenshot_2023-09-15_at_3.15.54_PM.png]]

  

**⇒ Distiction between User Space and Kernel Space**

This distincation is for virtual memory space.

**User Space:**

- **Definition:** User space is the area of memory where user-level applications and processes run.
- **Privileges:** User space has restricted privileges. Applications running in user space do not have direct access to hardware or critical system resources.
- **Activities:** User-level processes execute their code, handle user input, and perform most computing tasks within user space.
- **Isolation:** Each user-level process has its own isolated memory space, protecting it from interfering with other processes.

  

**Kernel Space:**

- **Definition:** Kernel space is the privileged area of memory where the operating system kernel resides.
- **Privileges:** Kernel space has elevated privileges, allowing direct access to hardware and critical system resources. Only code running in kernel space can perform certain privileged operations.
- **Responsibilities:** The kernel is responsible for managing system resources, handling hardware interactions, and providing services to user-level applications.
- **Isolation:** The kernel is a shared resource used by all processes. However, it is protected from direct access by user-level processes to prevent unauthorized or unsafe operations.