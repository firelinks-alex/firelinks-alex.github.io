*Pre-requisite*:
- Introduction to Computer Architecture
- Knowing some amount of C
- Some level of OS knowledge

*Scoring*:
- 70% projects (5 projects)
- 15% mid-tewrm
- 15% final

# 1. Introduction
## Overview of Hardware, software, OS and x86
**What is an Opearating System:** 

in a sentence, it is a software that's responsible for making it easy to run programs, or even allowing many programs to run together at the same time, allowing them to share memory, interact with external devices, and do all other fun stuff.

More specifically, 
1. It is a **virtual machine** because it **virtualizes** the hardware by transforming them into a easy-to-use virtual form of them self so the programs can share a physical resource together.
2. It is a **standard library** because it provides APIs to run programs, access memory, and access devices.
3. It is a **resource manager** because it manages hardware resource and make sure they get well-distributed to the applications using them.

**Virtualization** is a technique to **divide** the physical resources logically. Operating system achieves this by **abstracting away** the underlyinng complexity of resource segregation. For example, in concurrency the operating system divides a CPU to multiple programs by time sharing. In memory management, operating system divides a phisical memory into **virtual memory spaces** so that every program thinks they have the exclusive usage of the entire memory.

### Four themes of the textbook
1. Virtualizing CPU
2. Virtualizing Memory
3. Concurrency
4. Persistence: **file system**

## x86 implementation
- **EIP** (external instruction pointer) is incremented after each instruction completes
- **Instructions have different length**
- **EIP** is only modifiable via **CALL**, **RET**, **JMP**, and **conditional JMP** instructions

EFLAGs registers

**Registers are handled by compilers in the high level languages** so you don't have to manage them yourself. When our programs uses more data than a register file can handle we need to incorporate the **main memory (DRAM)**.

**Main Memory instructions**: MOV, PUSH, POP, etc. Most instructions can take one memory address (1 byte)

- `movl %eax, %edx`       | register mode: manipulation between two registers
- `movl $0x123, %edx`     | immediate: move a value to the register (`edx`)
-------------------------(slower)--------------------------------------
- `movl 0x123, %edx`      | direct: move the value in the main memory address to the register `edx`
- `movl (%ebx), %edx`     | indirect: treat the value in the `ebx` as an address and store the value in that address to the register `edx`
- `movl 4(%ebx), %edx`    | dispaced: move from the ebx by 4 and treat the value there as an address, then fetch the value and store it in the register `edx`

## Stack memory
What is a stack and why do we need it?

Let's say we have two functions:`main()` and `foo()` and we want to keep track of
1. what function is under execution.
2. what is the **scope** of a given function.
in order to execute them correctly.

**Context of a function**: all the variables and arguments given to a function. It lives in the stack, a portion of the memory will be dedicated to hold the context.

In the code below, when the funcion `foo()` executes `x++` it should increment the variable `x` in the `foo()`'s context, not the `x` in the `main()`'s context.
```c
main() {
    x = 20;
    foo();
}

foo() {
    x = 10;
}
```

Stack starts from a **higher address** and **grows lower** (downward). Everytime when a function call is made `%esp` grows smaller.

## Stack Registers:
- **EIP**: (external instruction pointer): which instruction in this stack to execute the next.
- **ESP**: (external stack pointer) : Initially located at the start of the stack region and grows downward as function calls. This register indicates **the location of the end of a stack**.
- **EBP**: to solidify the start point of this stack frame (and the arguments of a function are located above the EBP)
- **EAX**: to store the return value

Stack memory operations:
```asm
Example instruction:        What it does:
______________________________________________
pushl %eax                  subl $4, %esp
                            movl $eax, (%esp)

popl %eax                   movl (%esp), %eax
                            addl $4, %esp

call 0x12345                pushl %eip(*)           # function call at addres 0x123
                            movl $0x12345, %eip(*)

ret                         popl %eip(*)
```

## The goal of designing a Operating System
We want to design a operating system that is capable of taking physical resources and **virtualize** them. It handles concurrency issues so that multiple processes can run at the same time. It can store the files **persistantly**. While we are achieving these goals we also have to keep the following principles in mind:
1. The operating system should provide enough abstractions to make the system easy-to-use.
2. The OS should provide high performance, or minimize the overheads.
3. The OS should provide protection between applications, as well between the OS and applications.
4. The OS should provide high level degree of reliability (resiliency, fault-tolerant, error handling)
5. Energy efficiency
6. Security
7. Portability

# 2. The Abstraction: The Process (Operating Systems: Three Easy Pieces, Chapter 4)
**Virtualizing the CPU**: to create an illusion that the CPU is always  available to a process. A basic technique to achieve this is called **time sharing** of the CPU, where it executes the instructions from a process for some time and switch to execute the instructions from other processes.

To implement the virtualization of the CPU the OS needs both **low level machinery mechanisms** and **high level algorithms policies**.
- Mechanisms (How to): low-level methods or protocols that implement a piece of functionality. e.g. **context switch**
- Policies (which one): high-level algorithms for making decisions. e.g. **shcedulling**

## 4.1 The Abstraction: A Process
A process is the abstraction of the **machine state** of a running program. 
the **machine state** of a process is something the program can read or update, it consists of **address space**, **registers**, and the **files** associated with the program. The process' state alters when any of those elements changes. 
This also means during a **context switch**, the machine state of a process has to be stored properly 
for later restoration.

The information about a process is stored in a data structure like the one below:
```c
struct context
{
    int eip;    // instruction pointer
    int esp;    // stack pointer
    int ebx;
    int ecx;
    int edx;
    int esi;
    int edi;
    int ebp;    // stack base pointer
}

struct proc
{
    char* mem;  // start of process memory
    unit sz;    //size of memory
    
    enum proc_state state;  // current process state
    int pid;                // process id
    strcut proc *parent;    // parent process
    void *chan;             // if non-zero, sleeping on chan
    int killed;             // if non-zero, have been killed
    struct file *ofile[NOFILE]; // open files
    struct inode *cwd;      // current dir
    struct context context;
    struct trapframe *tf    // trap frame for the current interrupt
}
```
These data structures that employed by the OS to maintain the processes are called **Process Control Blocks (PCBs)**

## 4.2 Process API
OS often provides APIs so the underlying programs can take advantage of them. These APIs include but not limited to the following categories:
- process creation: `exec(), fork()`
- process destruction: `kill()`
- process synchronization: `wait(), join()`
- miscellaneous controls: suspend, resume...
- process status inquery

## 4.3 How a process starts
1. OS creates address space for the process and loads the program into the the space
2. OS initialize the run-time stack.
3. OS initialize I/Os including file descriptors
4. Jump to the `main()` routine of the program
5. Process starts

## 4.4 Process States
A process can have many states defined by an OS. A process transfers from a state to another state by schedulling or when certain event occurs in the system. For example:
```
process A (RUNNING) -> file_open() (BLOCKED) -> **file opened by the syscall handler** -> process A (RUNNING)
```

## 4.5 Data Structures
1. process list: all the processes
2. register context: for context switching
3. interrupt tables (IDT)
4. system call table / trap table
5. kernel code
6. interrupt and exception handler

These PCBs are normally stored in the **kernel memory space**, a memory region exclusively created for the kernel and it requires privileged access.

# 6. Mechanism: Limited Direct Execution
For virtualizing CPU, we employ a technique called **time-sharing the CPU**. 
This means, the kernel needs to regain the control back once a while for schedulling other processes. 
At the same time, we don't want this process to be inefficient.

**Limited Direct Control** is a technique that allowing a process to execute directly on the CPU (using the CPU directly) 
without an overwatching entity. 
It is *limited* because although the process' instructions natively execute on the CPU, it has to perform a **mode switch** 
for the priviledged instruction such as I/O operations.

## 6.1 How to enforce access during Direct Control
The access is enforced by introducing **system calls**, **user mode** and **kernel mode**. While in the user mode, 
a process can only execute unpriviledged operations such as memory read/writes. 
A process has to issue a **system call** to switch to the kernel mode where it can execute priviledged the kernel code.

A system call will let the execution jump to the kernel code that handles the system call. 
The handler usually first checks if the operation is valid for the process. 

System calls is a subset of **trap instructions** that stops normal flow of a program 
and jumps to the kernel context. Note that this is different from a **context switch** 
where the operating system **deschedules** the running program to schedule another program. 
By performing a trap instruction the user program will not be deschedulled. 
It will only jump to one of the kernel pre-prepared handlers and continue executing 
the kernel codes written in the handler.

When a process switching into kernel mode its state has to be saved (like before a context switch)
in order to restore the program's normal flow after finishing the priviledged operations.
The data structure used for saving the state is called **trap frame**. 
Later the program will start executing the code written in the trap handler. 
Once it completes it will issue a **return from trap (ret)** instruction that restores the program's original flow by 
looking up the saved trap frame and switching the process into user mode.  

On x86 the trap frame is saved in the **kernel stack**, 
a per-process memory region that stores **kernel stack frames**. 
We will come back to this later when we discuss **address space** later.

As you can see, an OS kernel isn't something that runs or overwatches the entire system independently.
It's rather *a part of a user process* -- whenever it needs to run the kernel code it performs a mode switch. 

## 6.2 How can OS regain control for schedulling other processes?
Next problem is how to let OS regain control while a user program is running so it can schedule other processes. 
There are two approaches we could adopt:
1. **Cooperative**: Set the user programs to send `yield()` system calls once a while. 
This system call will execute the handler code that deschedules the current user program and schedule another program to run.
2. **Non-cooperative**: the system utilizes external **hardware devices** like a **timer device** to 
issue interrupt signals like a **timer interrupt** periodically. 
When the CPU receives such a signal it let the current process jumps to execute 
the pre-defined handler code (again, raising the program's priviledge level to kernel mode) 
that deschedules the user program to schedulle other programs. 

# 7 - 10. Schedulling related
(WIP)

# 13. The Abstraction: Address Space
Address space is an easy-to-use abstraction of the physical memory. It provides both **simplicity** and **protection** to 
the processes. An address space of a program starts from address `0` and grows all the way to the `2^words` address.  
For example, in a 32-bit processor machine, the address space can range from `0x00000000` to `0xFFFFFFFF`. 
These memory addresses are called **virtual memory addresses**.

To a program, since it can address all the addresses in a 32-bit word system, 
it thinks it has the exclusive use of the computer memory. However, in reality, the OS will map these virtual addresses
in a program to the some physical memory addresses.
Therefore the virtual address `0x12345678` in a program isn't the same as the `0x12345678` in another program.

We will talk about the virtual-physical address translation in a later chapter. Here we just have to understand 
with help of address space, a program doesn't have to know what physical memory region it's allowed to read and write.
Every time it reads or writes to a virtual address the OS or corresponding hardware will be translating it to the 
correct physical address.

## 13.1 Address Space Segments  
An address space contains all of the memory state of a running program, 
that includes, the **code** of the program, the **stack**, and the **heap** segment segments. 
In most books, the code segment is located the bottom most location starting from the virtual address 0 and 
the stack segment is located at the top most location of the address space and grows downwards. 
The heap is often dipicted as it's located adjacent to the code portion and grows upwards.
Although this is a usaful model to think, it's not entirely true. You can arrange where these segments 
are located as long as it works. However, we will continue follow the convention for the simplicity.

## 13.2 The goals of designing a address space
1. The major goal of a **virtual memory (VM)** system is **transparency**. This means, the user program should not 
know it's utilizing a virtual memory. Instead, it should behave as it has its own private physical memory.
2. The virtual memory system should be **efficient** in both time and space. It shoud translate the addresses in a timely 
fashion and the it shouldn't occupy too much of the memory space.
3. The virtual memory system should **protect processes** from one another as well as the OS itself from the processes. 
When a process loads/writes to a memory it shouldn't affect other processes. If a process dies, it should not cause 
other processes to crash.

# 15. Mechanism: Address Translation
Like the practices in Limited Direct Execution the OS has to maintain both efficiency and control over memory. 
We want to make sure the virtual memory system runs light and fast as well as a process can only access 
its own memry region. Again, like in the design of LDE, we need some supports from the hardware in order to 
make this happen.

In the basic approach we only need a couple register support from the hardware. Later, once the VM system matures 
we will need more support from the hardware to implement **TLBs** and **page tables**. We will introduce these 
concepts later.

## 15.1 Hardware based address translation
Or in address translation in short, we weed the OS to keep track of what memory locations are free and occupied 
so it can resolve where in the memory the program's memory should be allocated. The hardware mechanism will help the 
program translate a virtual memory to its corresponding physical address by interposing each memory access.

## 15.2 Dynamic (Hardware-based) Relocation
Dynamic Relocation is also known as **base-and-bound** technique. This solution needs a **base** and **bound** registers 
support from the processor (they are in the **Memory Management Unit (MMU)** of the CPU).

When the OS starts a program, it look up where in the physical memory the program can be allocated continuesly and 
set the *base* register to that value. Since an address space is also continues, the hardware can treat any virtual 
memory from the program as the **offset** to the base register value. Therefore, `physical_addr = base + virtual_addr`.

The OS also puts the maximum physical address the process can access to the **bound** register so the hardware can 
detact if the memory the process is trying to access is valid or not.

The downside of this approach is obvious: the physical addresses needed to allocate the program has to be continues.
It means this method only work well when the address space size is smaller than the physical address space. Otherwise 
the memory space won't be efficient for the next program to run (maybe we have to allocate a large **swap** partition in 
order to fit many processes.We will discuss this topic later in this guide)





# Computer Systems Book:
- Chapter 1: A fast introduction of the computer systems by tracing a simple "hello world" program.
- Chapter 2: Computer Arithmetic 
- Chapter 3: Assembly code (I32, x86-64)
- Chapter 4: Processor architecture
- Chapter 5: Performance
- Chapter 6: Memory hierarchy
- Chapter 7: Linking
- Chapter 8: Control flow
- Chapter 9: Virtual memory
- Chapter 10: Unix I/O
- Chapter 11: Networking
- Chapter 12: Concurrency


# Chapter 1. A tour of computer systems
# 1.1 Information is Bits + Context
When we write a `hello_world.c` program we are writing it in **plain text** just like other ASCII characters in a `.txt` file. These *characters* are internally `bits` in computer systems.

However, we don't treat `hello_world.c` as same as a plain text file, instead we interpret it following the C **syntax**. This special **view** of the file is called the **context**. Regardless of what storage (e.g. hard drive, memory, registers) stores the bits, the *meaning* of the bits changes when the context changes.

During compilation of a C source file we often observe multiple context changes:
- `hello.c` -> pre-processor -> `hello.i` (process the `#` directives. e.g. insert the header files indicated by the `#include` and process macros defined by `#define`)
- `hello.i` -> compiler -> `hello.s` (generate the assembly code by following the C-assembly conversion rule)
- `hello.s` -> assembler -> `hello.o` (generate **machine language instructions** as know as `relocatable object files` by interpreting the assembly code)
- `hello.o` -> linker -> `hello` (generate the `executable object file` by linking the necessary `relocatable object files`)

## 1.3 What are programs/processes
A program is a set of written instructions to a processor. The instructions can involve input/output works, memory management, and arithmetic calculations. Therefore, a process is an abstraction of I/O, memory, and CPU.

## 1.4 How `hello` gets executed in a shell
Initially, the shell program is *running* and waiting the user's input. Note that the term **running** here means the instructions written in the shell program's executable file are being executed in the CPU. More percisely, it is the instructions checking the user's input repeatatively.  

As user starts typing `./hello` the command line, the shell insturctions store each character into a CPU register, and then stores it in the main memory.

When the user hits ENTER on the keyboard, the shell knows he finished typing (by some mechanisms like detecting the EOF character etc.). Then the shell asks **Dynamic Memory Access (DMA)** to load the bytes in the `hello` executable object file from the hard drive to the main memory.

**Dynamic Memory Access** is a method that allows an I/O device to send or receive data directly from the main memory without bypassing the CPU.

## 1.5 Abstractions
- I/O devices (hard disk, network) are abstracted as **Files**
- The physical memory and I/O devices (files) are collectively abstracted as **Virtual Memory**
- A processor is abstracted as **instruction set architecture** 
- A running program (set of written instrcutions) is abstracted as a **process**.

### 1.5.1 Threads
A thread is an abstraction to a group of instructions. A program consists of many instructions, and a thread bundles multiple instructions together as a single (*schedullable*) **execution unit**. This makes concurrency more light weight.

### 1.5.2 Virtual Memory
Virtual memory is an abstraction to the main memory and I/O devices. It gives a program an illusion that it owns the entire memory exclusively. This unifrom view of memory is called **Virtual Address Space**. 

The virtual address sapce seen by each process consists of a number of well-defined areas. From the top (large address) to bottom (small address): 

- **kernel vitual memory**: Kernel is the part of the operating system that always resides in the memory. Application programs are not allowed to read of write the contents of this region or to directly call the functions defined here.
- **Stack**: user function calls are implemented here
- **Shared libs**
- **Heap**
- **Program code and data** 

### 1.5.3 Files
**A file is a sequence of bytes**. Every I/O device, including disks, keyboards, displays and even networks is modeled as a file. All input/outputs in the system are performed by reading and writing files, using a small set of system calls known as Unix I/O.

## 1.6 Instruction-Level Prallelism
Modern processors can execute multiple instructions at one time. This is done by **pipelining**, a technique that performs the steps required to execute an instruction simutaniously, with help of the architectural characteristics of a processor (**stages**).

# Chapter 2. Representing and Manipulating Information
## 2.1 Information Storage
**Bytes** are the smallest addressable unit of memory.

The value of a **pointer** in C-wether it points to an integer, a structure, or some other program object-is the vitual address of the first byte of some block of storage. When a pointer is declared in a C source, the compiler also associates the *type* information with the pointer, so it can generate different **machine code** to access the value stored at the location appropriately. It is important to note that the actual **machine code** generated doesn't preserve the type information. It still treats each **program object** as a block of bytes and the program itself as a sequence of bytes.

# Chapter 3. Machine-Level Representation of Programs (p.154 - p.332)
Computers execute **machine code**, a sequence of bytes encoding the low-level operations that manupulate data, manage memory, read and write data on storage devices, and communicate over networks.

A compiler generates machine codes based on various rules that a language defines. For example, *gcc* compiler negerates **assembly code**, a textual representation of machine code, from the C source file. It then invokes both **assembler** and **linker** to generate the executable machine code from the assembly code.

Using high-level languages and compilers to write programs brings several benefits:
- With help of a compiler we can ensure type errors don't happen as frequent as using a machine code or low-level language
- We can program compilers to generate professional level assembly code so the higher language author doesn't have to worry about too much.
- We can program compilers to generate different machine code to different architecture/OS while keeping the higher language the same
  
However, we still want to be able to read assembly code due to several reasons:
- We want to understand the execution model of a system
- We want to identify the optimization capabilities of a compiler and analyze the underlying inefficientcies in the code.
- For debugging espacially concurrent programs
- Analyze vulnerabilities

In this chapter we will learn two particular assembly languages, IA32 (32bits) and x86-64 (64bits) and see how C programs get compiled into these forms of machine code. By doing this we will understand the transformations typical compilers make in converting C into machine code.

Here are couple things we want to explore by learning these assembly codes:
- Relationship between C, assembly, and machine code, How data is represented and manipulated.
- How control statements like `if`, `while`, and `switch` statements are implemented
- How the procedures are implemented. Specifically how the program maintains a run-time stack to support passing data and control between procedures, as well as for storage for local veribles.
- How data structures such as arrays, structs, and unions are implemented
- What is out-of-bounds memory references and the vulnerability of system to buffer overflow attacks.
- GDB debugger

As a bonus knowledge we also want to show
- How to write entire functions in assembly code and comebine them in the linking stage.
- How to use gcc's support for embedding assembly code directly within C programs.

## 3.1 Historical backgrounds
From intel's first processor, 16-bit 8086 to the latest Core i7, each successive processor has ben designed to be backward compatible. This introduced many starnge artifacts in the instruction set due to this evolution heritage. 

Over the years, the several companies have produced processors that are compatible with Intel processors, capable of running the exact same machine-level programs. Cheif among these is AMD (Advanced Micro Devices).

```
Flat addressing: the entire memory space is viewed by the programmer as a large array of bytes.
```

The default invocation of gcc will generate code for i386. In order to produce more recent instruction sets, we can simply provide more arguments when invoking the compiler.

## 3.2 Program Encoding
Let's say we have two C files `p1.c` and `p2.c`. We can compile them on an IA32 machine using a Unix Command line: 
```
unix> gcc -O1 p p1.c p2.c
```
In this command the option `-O1` (not number 0, it's "O" as in orange) instructs the compiler to apply level-one optimization. In general, increasing the level of optimization makes the final program run faster, but at a risk of increased compilation time and difficulties running debugging tools on the code. As we will also see, invoking higher levels of optimization can generate code that is so heavily transofrmed that the releationship between the generated machine code and the original source code is difficult to understand. We will start from the level-one optimization as a learning tool and then see what happens as we increase the level of optimization.

When we call `gcc` on the command line it actually invokes a sequence of programs to turn the source code into executable code:
1. C *preprocessor* is invoked  to expand the source code to include any files specified with `#include` commands and any macros specified with `#define` declarations.
2. C compiler generates assembly code for the two files `p1.c` and `p2.c` having names `p1.s` and `p2.s`
3. Assembler convers `.s` files to `p1.o` and `p2.o` **object-code** files. Object-code is one form of machine code-it contains binary representation of all of the instructions, but the address of global values are not yet filled in.
4. Linker merges these two object code files along with code implementing library functions (e.g `prinf`) and generates the final **executable code** file `p`. Executable code is the second form of machine code and it is the exact form of code that's executed by the processor.

### 3.2.1 Machine-Level Code
The abstract model used to describes a type of processor is **instruction set architecture (ISA)**. It defines the followings: 
- the format and behavior of a machine-level program running on the processor
- the processor state
- Instruction format, and the effect each of these insturctions will have on the processor's state.

```
Programming Model: あるプログラミング言語あるいはモジュール、ライブラリー、ランタイムにおいて、あるプログラミング要素に対するシステムの振舞い方である。これは後程アプリケーション（出来上がったモノ）とその実現方法（作り方）との繋がりの役割を果たす。
またプログラミングモデルを勉強するより、あるプログラミング要素に対してシステムは具体的にどう反応するかを判明することができる。
```

These items are visible in assembly code but are hidden from C programmers:
- *program counter* or "PC" is called `%eip` in IA32. It indicates the next instruction to be executed
- The integer *register file* contains eight named locations storing 32-bit values. They can hold addresses (C pointers) or integer values. Some registers are used to keep track of critical parts of the program state, while others are used to hold temporary data, such as the local variables of a procedure, and the value to be reutned by a function.
- The condition code registers hold status information about the most recent executed arithmetic or logical instruction. These are used to implement conditional changes in the control or data flow, such as required to implement `if` and `while` statements.
- A set of floating-point registers store floating-point data.

These are the items meaningful in C code but lose their meaning once in the assembly form:
- data types; machine code views the memory as a large byte-addressable array.
- arrays and structures; these are represented in machine code as contiguous collection collections of bytes.
- scalar data types; machine code doesn't distinguish int or unsigned int, different types of pointers, pointers and integers. They are all the same to machine code.

Additionally, the **operating system** enforces only limited subranges of virtual addresses are valid. This can prevent the programs from accessing the memory portions aren't assigned to them. The operating system manages this virtual address space by translating virtual addresses into the physical addresses of values in the actual processor memory.

A single machine instruction performs very elementary operation like:
- add two numbers stored in registers
- transfer data between memory and a register
- conditionally branch to a new instruction address
The compiler must generate sequences of such instructions to implement arithmetic expression evaluation (`int a = b + c;`), loops `for`, procedure calls and returns (`int function() {}`).

### 3.2.2 Code Examples
```
The assembly code generated by a compiler changes changes as gcc version or command-line option changes. Do not expect these assembly code stays unchanged. It's important to learn how to map the generated code back to the original constructs.
```

Suppose we have the following C code `code.c`
```c
int accum = 0;
int sum(int x, int y) {
    int t = x + y;
    accum += t;
    return t;
}
```
To see the assembly code generated by the C compiler, we can use `-S` option on the command line:
```
unix> gcc -O1 -S code.c
```
```
sum:
    pushl %ebp // push the contents of register %ebp onto the program stack
    movl %esp, %ebp
    movl 12(%ebp), %eax
    addl 8(%ebp), %eax
    addl %eax, accum
    popl %ebp
    ret
```
In the assembly code above we no longer see the variable `x` and `y`. But we still see the reference to the global variable `accum`, since the compiler has not yet determined where in memory this variable will be stored.

If we add `-c` command-line option, GCC will both compile and assemble the code generating *object code*:
```
unix> gcc -O1 -c code.c
```
This generates an object-file `code.o` that is in binary format and hence cannot be viewed directly.
The hexadecimal representation of `code.o` is a 17-byte sequence:
```
55 89 e5 8b 45 0c 03 45 08 01 05 00 00 00 00 5d c3
```
This object-code corresponding to the assembly instructions in `code.s`. A key lesson to learn from this is that the program actually executed by the machine is simply a sequence of bytes encoding a series of instructions.

With help of **disassembly** programs like **OBJDUMP**, we can convert a object file to its assembly format.
```
unix> objdump -d code.o
```
```
1 00000000 <sum>:
    Offset Bytes                Equivalent assembly language
2   0:      55                  push %ebp
3   1:      89 e5               mov %esp,%ebp
4   3:      8b 45 0c            mov 0xc(%ebp),%eax
5   6:      03 45 08            add 0x8(%ebp),%eax
6   9:      01 05 00 00 00 00   add %eax,0x0
7   f:      5d                  popl %ebp
8   10:     c3                  ret
```
Here are couple take aways from the disassembled code above:
- An IA32 instructions length can vary from 1 to 15 bytes. The instruction encoding is designed so that a more frequently used ones and the ones have fewer operands require smaller number of bytes.
- For a given instruction, there is a unique decoding of bytes from a starting point. For example, only `push %ebp` can start with byte 55.
- The global variable's identifier `accum` is replaced with a memory address `0x0`. However, it doesn't look like a *real* run-time address.
- The disassembler determines the assembly code purely by the byte sequences in the machine-code file. It does not require access to the source or assembly-code version of the program.
- The disassembler uses slightly different naming convention for the instructions than does the assembly code generated by GCC. In our example, it has omitted suffix `l` from many of the instructions. These suffixes are **size designators** and can be omitted in most cases.

Generating the final executable file requires running a linker on the set of object-code files, one of which must contain a function `main`. Let's say we have a `main` function written in the `main.c` file:
```c
int main() {
    return sum(1, 3);
}
```
We may generate an executable file `prog` by specifying the object file `main.c` needs:
```
unix> gcc -O1 -o prog code.o main.c
```
Although the command above will give a warning `implicit declaration of sum` the code compiles just fine. This is because the `main` function calls `sum` without properly `#include` in the beginning of the file. The compiler is able to resolve the `sum` by looking at the object file we provide it.

If we disassemble the `prog` executable file we will notice couple things:
- two assembly codes look almost the same with subtle differences
- the memory addresses and offsets look different than the disassembled code of `sum` having more realistic addresses.
- the variable `accum` has been replaced by a real address.
- 
