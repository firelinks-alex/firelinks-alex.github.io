Pre-requisite:
- Introduction to Computer Architecture
- Knowing some amount of C
- Some level of OS knowledge

Structure:
- 24 lectures

Scoring:
- 70% projects (5 projects)
- 15% mid-tewrm
- 15% final

To see and run
- Run make
- cd /build directory
- run tests: make check
- clean the /build: make clean
- run single test case: make tests/threads/priority-donate-next.result VERBOSE=1

# Introduction and Demos:
## Overview of Hardware, software, os and x86

- Application
- Libraries (printf/glibc)
- OS
- HW

Purpose of an operating system:
- abstract hardware for performance and flexibility
- allow multiple users to run varietiy of application
- multiple hardware resources between multiple applications and users
- isolate applications
- allow sharing among applications
- provide high performance

You want isolation but you also want sharing which is contradictory. This is one of the chanllenges of designing a good operating system.


Why study operating systems:
- Very high performance on a hardware, extract the most performance of a hardware.
- To beter understand what happens underneath the hood (understanding the principle)

Abstractions provided by the OS:
- for computation: process
- for memory: address space
- for storage: files
  
Why is designing an os hard
- must be efficient but also portable (more complex = more bugs)
- must be powerful but simple
- feature interact: like forking a process and open files

## x86 architecture
e.g. jaguard board
How do you program the processor: x86 instruction set: the same series of 0s and 1s to the processors means different

Abstract model: input/output <- CPU <-  Main memory (N words of B bits)
 
Memory holds instructions and data (argument)

x86 implementation
- EIP (external instruction pointer) is incremented after each instruction
- Instructions have different length
- EIP is modified by CALL, RET, JMP, and conditional JMP instructions


EFLAGs registers

Registers are handled by compilers in the high level languages
However, when our programs uses more data than the registers can handle we need to incorporate memories.

Memory instructions: MOV, PUSH, POP, etc
Most instructions can take a memory address

- register mode
- immediate
- direct
- indirect (C pointer)
- dispaced

Stack memory + operations

What is a stack and why do we need it:
There are different operations we want to do

We have `main()` and `foo()`, We need to keep track of
1. what function is under execution
2. what is the written scope of a given function

```
main() {
    x = 20;
    foo();
}

foo() {
    x = 10;
    x++;
}
```

Context for a function: all the variables and arguments given to the function

It lives in a stack, a portion of a memory that holds the context of the function

Stack starts from a higher and grows lower (downward): #1000 -> #900  
which means every time you call a function the number(add) becomes smaller

- ESP: external stack pointer : where is the end of a stack
- EBP solidify the start point (and the arguments of the function are located above the EBP)

Computer Systems Book:
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
unix> gcc -01 p p1.c p2.c
```
In this command the option `-01` instructs the compiler to apply level-one optimization. In general, increasing the level of optimization makes the final program run faster, but at a risk of increased compilation time and difficulties running debugging tools on the code. As we will also see, invoking higher levels of optimization can generate code that is so heavily transofrmed that the releationship between the generated machine code and the original source code is difficult to understand. We will start from the level-one optimization as a learning tool and then see what happens as we increase the level of optimization.

When we call `gcc` on the command line it actually invokes a sequence of programs to turn the source code into executable code:
1. C *preprocessor* is invoked  to expand the source code to include any files specified with `#include` commands and any macros specified with `#define` declarations.
2. C compiler generates assembly code for the two files `p1.c` and `p2.c` having names `p1.s` and `p2.s`
3. Assembler convers `.s` files to `p1.o` and `p2.o` **object-code** files. Object-code is one form of machine code-it contains binary representation of all of the instructions, but the address of global values are not yet filled in.
4. Linker merges these two object code files along with code implementing library functions (e.g `prinf`) and generates the final **executable code** file `p`. Executable code is the second form of machine code and it is the exact form of code that's executed by the processor.

### 3.2.1 Machine-Level Code
The abstract model used to describes a type of processor is **instruction set architecture (ISA)**. It defines the following: 
- the format and behavior of a machine-level program running on the processor
- the processor state
- Instruction format, and the effect each of these insturctions will have on the processor's state.

These items are visible in assembly code but are hidden from C programmers:
- *program counter* or "PC" is called `%eip` in IA32. It indicates the next instruction to be executed
- The integer *register file* contains eight named locations storing 32-bit values. They can hold addresses (C pointers) or integer values. Some registers are used to keep track of critical parts of the program state, while others are used to hold temporary data, such as the local variables of a procedure, and the value to be reutned by a function.
- The condition code registers hold status information about the most recent executed arithmetic or logical instruction. These are used to implement conditional changes in the control or data flow, such as required to implement `if` and `while` statements.
- A set of floating-point registers store floating-point data.





