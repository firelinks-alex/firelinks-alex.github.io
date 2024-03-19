# Chapter 2. Introduction to Operating Systems (OS Basics)
- **Von Neumann model**: the processor **fetches** an instruction from memory, **decodes** it, and **executes** it.
- **Primary goal** of designing an OS: to make it correct, efficient, and easy to use.
- **Virtualization** is a technique to **divide a single physical resource** to many virtual resources to create an illusion to the programs programs of infinite number of resources.
- An OS is
    1. a **virtual machine** because it virtualizes physical resources.
    2. a **standard library** because it provides easy-to-use APIs.
    3. a **resource manager** because it manages them in order to achieve efficientcy and security.
- An OS virtualizes CPU by **time-sharing** the CPU between the processes, and it virtualizes memory by creating a **private virtual address space** for each processes.
- An OS **does not virtualize the hard disk** because it assumes the files are to be shared between multiple processes.
- An OS introduces **system calls** to facilitate the ease of use and security.
- Early OS was nothing but a standard library that provides **procedure calls** to the processes. Later it involved to include protection mechanism named **system calls** to prevent the processes to again access to the protected resources
- When a **system call** takes place the **trap instruction** set up in the hardware,  **trap-handler**, is actuated, transferring the control over the OS. And at the same time, raises the **privilege level** to the **kernel mode**. Once the protected resource is accessed by the OS, it executes **return-from-trap** instruction to transfer the control back to the application, and at the same time, lowering the privilege level to the **user mode**.
- **multiprogramming**: multiple processes run simutenously in a computer.
- With multiprogramming there comes problems of **memory protection** and **concurrency**.

# Chapter 4. The abstraction: the process
- An OS implements **time-sharing** of CPU by employing both a **low-level mechanism** named **context-switch** and high-level algorithms named **schedulling policies**.
- As a counterpart of **time-sharing**, we **space-sharing** that devices a physical resource into many pieces. For example, hard disk. Once a block is assigned to a file, it won't be assigned to another file until the file gets deleted.  
- **mechanisms** are low-level methods/protocols that implement a functionality.
- **policies** are algorithms that make some kind of decisions.
- A process's **machine state** constitues the necessary parts to **descibe a process**:
    - **address space** (**memory**): the memory that the process can address. 
    - **registers** (**CPU**): program counter, stack pointer, stack frame pointer etc.
    - **I/O devices**: opened files
- **Process API** is a set of tools an OS provides for creating, destroying, waiting, miscellaneous controls, and status inquery a process.
- How OS creates a process:
    1. OS allocates a memory space for the process as its address space
    2. OS loads the program text and data stored in a persistant storage like a disk to the program's address space.
    3. OS initializes the run-time stack
    3. initializes heap and I/Os
    4. jumps to the main() routine of the program, and program starts
- Process state:
    - **RUNNING**: this process is currently running.
    - **BLOCKED**: this process is blocked from running and needs some type of event to happen in order to schedule it again. 
    - **READY**: this process is ready to be schedulled.
    - **zombie state**: the process is existed but not cleaned up
- A Running process can be descheduled at any time, and a Ready process can be scheduled at any time. A Blocked process needs some type of event to occur in order to be scheduled again.
- Data structures used for managing processes
    1. **process-list**: all-list, blocked-list, ready-list
    2. **register context**: to hold the registers of a process
    3. **PCB: process control block**: start of memory, size of the memory, bottom of the kstack, context, state, parent, interrupt frame.
 
 # Chapter 5. Process APIs
 - fork(): the parent process receives the PID of the child process. the child receives **ZERO**. The child process does not start from main(). **Instead, it starts from where it fork() was called**. However, it has its own address space, register files, and I/O list. The output of fork() is non-deterministic because either of them can be schedulled at anytime.
 - wait(): waits a process to **exit**.
 - exec()/execvp(): it runs the process specified in the argument. However, it overrides the code segment of the current process with the code of the the process. it's like swapping the process in a address space with another process.

 # Chapter 6. Limited Direct Execution (LDE)
 - **Limited Direct Execution** was introduced because we want both **efficiency** and **control**.
    - a process directly runs on a CPU, without any middleware, we can maintain **efficiency**.
    - by differenciating **kernel mode** and **user mode** we can maintain high security/control
- **Limited** means an OS takes advantages of:
    1. **system calls** (Cooperative):
        - when a program initiates system call, it executes a **trap instruction**
        - A trap instruction is handled by a dedicated **trap handler**, and trap handlers are stored in the hardware by the kernel in form of **trap table** (system-call number to handler mapping) during the boot process.
        - The processor pushes the processes' register information on to the **kernel stack** (in case of x86 processor), a restricted region in the address space that can only be accesses in **kernel mode**. At the same time, it resumes the OS's trap handler.
        -  Once trap handler finishes the work, it executes **return-from-trap** instruction that pops the register information stored in the process's kernel stack and jumps to the where it was left off.
        - there is special system call: **yield** system call that does nothing but giving the control back to the OS.
    2. **Timer interrupt** (Non-Cooperative):
        - OS initializes the timer device in the hardware to issue **interrupt call** every once a while.
        - OS sets up timer, **interrupt handler** in the **handler table**
        - Once timer ticks, the hardware store the registers in the kernel stack then invoke the interrupt handler.
- After OS takes back the control:
    - the OS decides what process to run next based on the schedulling policies
    - while the OS executing the handler code, it may **disable the interrupt**, so it can finish whatever it's about to do. It enables the interrupt after it finishes the task.
- How **context switch** works:
    1. Process A is running
    2. timmer interrupt/system call happens, the hardware saves the Process A's regsiters to its kstack(A), and invokes the trap handler giving control back to OS.
    3. kernel saves the Process A's registers from its kstack(A) in a form of PCB in the kernel space.
    5. Kernel restores the process B's registers from kernel space and points the **stack pointer** to the B's kstack(B).
    6. OS issues **return-from-trap** instruction
    7. the hardware pops the registers from the stack pointer it pointed to, which is kstack(B)
    8. process B resumes.

# Chapter 7. Schedulling


# Chapter 13. Address Space
*Every address that a user program uses is a **virtual address**.*

In the early days, there was just two spaces in the physical memory. One for the OS to store its static code, another for the current running process. At this period of time OS was nothing but a library.

Later, for implementing **multiprogramming/time-sharing**, OS started to **isolate** the running programs memory, so they can't read/write other programs memory. But, how does an application knows what address it can write to? For streamlining this, OS developers came up with **Address Space to abstract** the view of the memory in the perspective of a program.

- An Address Space goal has these in mind:
    1. **simplicity/ease of use**: an application developer does not need to know which physical address to use/avoid.
    2. **transparency/abstraction**: an application developer does not have to know who the actual physical memory is laied out/implemented.
    3. **protection/isolation**: no process should be able to access the memory address they aren't allowed to.


<span style="color:red">**All the virtual addresses spaces starts from address 0**</span>.

- **An Address Space(AS) contains all the memory state of a program**
    - **code**: code, static data of the program
    - **heap**: dynamically(run-time) allocated memory
    - **stack**: function call chains

# Chapter 14. Memory APIs
- stack memory is automatically handled by C compilers. Thus, we don't have to explicitly create/delete them.
- heap is allocated using `malloc(sizeInByte)` function, and it returns a `void *` pointer which will need to be casted explicitly.
- `malloc()` fails with `NULL`
- `sizeof()` operator can get size in bytes
- when allocating heap memory for a string, remember to use `strlen(s) + 1` for its size since the string needs **end-of-string** character. Do NOT use `sizeof()`


# Chapter 15. Address Translation
Like in the CPU virtualization, we need to maintain both efficiency and control over the memory virtualization. **Control in memory** implies that the OS ensures no application is allowed to access any memory but its own.

- OS keeps track of:
    - used and free memory
    - maintain control over how memory is used
    - set up hardware for translation
- **hardware-based address translation**: Hardware transforms each memory access. (virtual -> physical and vice versa)
    - **Dynamic (run-time) hardware based relocation (a.k.a. base-and-bounds**): invented in 50's, we need two registers in the CPU/**Memory Management Unit (MMU)**
        1. when a process starts, OS decides where in physical memory it should be loaded and sets the **base** regsiter to that value (kernel mode).
        2. The process issues a fetch instruction against a virtual address (user mode/trap instruction)
        2. Hardware checks if the virtual address is in **bound**
        3. If it's out-of-bound then the OS prepared exception handler gets triggered. (trap table)
        3. Later, the hardware calculates `physical_address = virtual_add + base` (because virtual address starts from zero, so it works as an offset) to find the physical memory.
        4. Since the relocation takes place during run-time (after the program has started), it is called **dynamic relocation**. It is also possible to relocate the physical memory after started. (the process must be descheduled)

- With this method, since there are only a pair of base-and-bound registers, OS has to store the registers in memory in case of a context switch.

# Chapter 16. Segmentation (to avoid big free space)
Address spaces starts from address 0 and grows to certain size. With base-and-bound technique though, it can be **wasteful** to the physical memory because there can be unused space between code/heap/stack since the technique maps the entire address space at once. It is also impossible to map a address space that's larger than the physical space. **Segmentation** is invented for solving these problems. 

Segmentation means **contiguous portion** of the <u>**address space**</u> of a particular length. (it's a virtual address space thing!)

- we need to use three pairs of **base-and-bound** registers to store code, heap, stack segments. **Plus, additional protection bit to indicate if the region is shared**.
- those three segments no longer have to have only one **base** value. They can be located anywhere in the memory, **independently**.
- **sparse address space**: <u>address space</u> that has lots of free space.
- **coarse-grained** segmentation: has a few logical segments
- **fine-grained** segmentation: has a lot of logical segments -> needs hardware support: **segment table**

- segmentation problems (p.162):
    1. External fragmentation: the physical memory is occupied by lots of small segments, creating unsed spaces between them. Those unsed spaces, though, too small to be utilized.
    2. For fixing external fragmentation, OS can relocate the **Non-compacted memory to Compacted memory** However, copying data from one space to another introduces OS overhead.

# Chapter 18. Paging (to chop address space into fixed size)
With segmentation, we had external framentation. In order to combat this, we use alternative approach named paging.

- Paging is a practice of chopping the address space to fixed-sized units called **Pages**.
- Similarly, we divide physical memory to fixed-sized chuncks named **Page frames**.
- Segmentation: variable-sized logical blocks represents code, heap, stack...
- Paging: fixed-sized units, what they stores doesn't matter
- Advantage:
    1. **Flexibility**: we no longer care about what logical block a page stores. We don't have to make assumption about what direction the segment grow (e.g. stack). With well-developed paging, we can abstract those details and unify the translation process.
    2. Simplicity: the OS only has to keep track of free pages, not entire memory.
- Page to Page Frame mapping is stored in **page table**. This is a **per-process** data structure.
    - this is because, let's say we have P1 and P2.
    - P1 has VP1 mapped to PF2, and P2 has VP1 mapped to PF3.
    - if we don't store the page table per-process base, OS wouldn't know which PF the VP1 is pointing to.
- Virtual address = Virtual page Number (VPN) + offset
- address translation is done with `FindPageFrame(VPN) + offset` 
- As **page table** can get very large. We can't store them in **MMU**. **We store them in memory**.
- An entry in the table is called **page table entry(PTE)**
- the simplest form of a page table is **linear page table**, which is just an array. It is indexed using VPN for looking up PTE.
- In a PTE, we have number of bits
    1. valid bit
    2. present bit: it it on physical memory or disk (swapped out)?
    3. dirty bit: is it modified
    4. reference/access bit: has it been accessed?, is it a poppular page? -> keep in memory. (**Page-replacement**)
- paging problem: 
    1. too slow it requires extra memory reference for fetching the page table.
    2. page tables can be large

- **Page table base register**: contains the physical address of the starting location of the page table.

# 19. Faster Paging Translation: TLB (Translation Lookaside Buffer)
A TLB is a hardware cache. **address translation buffer** that stores virtual to physical address map.

How TLB works:
1. A process issues access to a memory address
2. Hardware checks if the virtual memory is in TLB.
3. If it finds the entry, it's called **TLB hit**. It translate the memory and append offset to access the memory.
4. If it doesn't, it access the **page table** to look for the mapping. Later, it either creates a new entry in TLB if the address is valid (exists in the page table), or it raise an exception.

- We focus on increasing the **TLB hit rate**
- **temporal locality**: quick re-referencing of memory items in time would increase TLB hit rate
- Data structures that support **special locality** like an array benefits the most from TLB.
- In a CISC hardware system, it's the hardware handles the **TLB miss**. **hardware-managed TLBs** uses **multi-level page table**
- In a **RISC** hardware, OS handles it (software-managed TLBs) -> trap instructions
