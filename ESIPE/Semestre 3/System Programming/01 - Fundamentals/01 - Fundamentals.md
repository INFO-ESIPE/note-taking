[[System Programming]]
****
**Table of Contents**
```table-of-contents
```

****
## Operating System - In a nutshell

As briefly explained [[01 - Introduction aux OS|here (in French)]], an Operating System is a special layer of software that provides application software access to hardware resources.
	*See it as an intermediate layer between applications and hardware*
![[os.png]]

That body of software is called the operating system as it is in charge of making sure the
system operates correctly and efficiently in an easy-to-use manner. It allows to implement several handy mechanisms:
- Abstraction of complex hardware devices
	*Machines uses different hardware, the OS is here to standardise all of this to facilitate how applications interact with those components*
- Protected access to shared resources
	*Memory, CPU, storage...*

All of this is achieved via several mechanisms, such as [[01 - Threads (EN)#Scheduler|scheduling and concurrency]], transactions, security (authentication, authorisation), and communication amongst logical entities (Inter-Process Communication, Signals ...).

### Virtualisation

One of the main characteristics of OSes is the technique of **virtualisation**. The OS takes a physical resource (such as the processor, or memory, or a disk) and transforms it into a more general, powerful, and easy-to-use virtual form of itself (which is mandatory if we want to share resources between applications).

The OS is like an **illusionist**: It provides clean, easy-to-use abstractions of physical resources, and will **trick each application into believing it has the entire machine for itself** (it's own private memory space, a seemingly infinite number of CPUs...)

Let's focus on memory virtualisation. The model of physical memory presented by modern machines is very simple: It's simply an array of bytes.
To read or write in memory, one must specify an address to be able to access
the data stored/to store there.
	*Memory is separated into big separated areas, as explained [[04 - Mémoire#Mémoire pour l'assembleur|here (in French)]] or [here (in English)](https://youtu.be/LfaMVlDaQ24?t=39706).*

Let's take the following code, and run it two times (so it illustrates the point we aim at):
```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include "common.h"

int main(int argc, char *argv[]) {
	int *p = malloc(sizeof(int)); // allocates on the heap
	assert(p != NULL);
	printf("(%d) address of p: %08x\n", getpid(), (unsigned) p); // prints addr 
	*p = 0; // puts an integer (0) in the allocated memory slot
	while (1) {
		Spin(1); // 1 second delay
		*p = *p + 1; // Increments the value stored at the address held by p
		printf("(%d) p: %d\n", getpid(), *p); // Prints the value pointed by p
	}
	
	return 0;
}
```
```bash
> ./mem &; ./mem &
[1] 24113
[2] 24114
(24113) memory address of p: 00200000
(24114) memory address of p: 00200000
(24113) p: 1
(24114) p: 1
(24114) p: 2
(24113) p: 2
(24113) p: 3
(24114) p: 3
(24113) p: 4
(24114) p: 4
```

Something "strange" happens: both our programs (`id 24113` and `id 24114`) have allocated the exact same memory address (`0x00200000`), which shouldn't be possible, right?
You might have guessed it already by reading this chapter's name, but each running program has its own private memory, instead of sharing the same physical memory with other running programs. The OS is **virtualising memory** so each process can have it's own **private address space** that is mapped to the real memory under the hood.

> [!info] *(Hardware) Virtualisation is a concept that extends to virtual machines and beyound, more details [[01 - Hardware Virtualization|in this class]].*

### Program

How do we get programs running on the OS ? In a nutshell, when you launch the binary file corresponding to the program we want to execute (double click it from the GUI, or launching it from a terminal with `./`), all this machine code is loaded from the disk to the computer's memory.
	*So, all the bits composing our program (like `spotify.exe`) are collected from the disk (containing the file itself) and loaded into a dedicated part of the RAM called the "machine code area"*

All this binary corresponds to instructions. And that's what a running program is doing: it executes a large quantity of instructions one by one:
1. The processor fetches an instruction from the machine code area
2. It decodes it (figures out which instruction this is)
3. It executes it (add two numbers together, access memory, jump to a function ...)

> [!info]
Note that an instruction is supposed to be doing one simple task, unlike our high-level programming language code which does several things per line. This is a behaviour you clearly notice when you look at those instructions in the assembly format.

What we described above is the basics of the Von Neumann model of computing:
![[neumann.png]]

### Process

The OS encapsulates a running program in a process. Each process has its own address space (as we briefly explained [[ESIPE/Semestre 3/System Programming/01 - Fundamentals/01 - Fundamentals#Virtualisation|here]]), at least one thread where the execution flow really happens, and associated system states (opened files, opened network connections ...).

> [!info] How to uniquely identifies processes running on the OS?
> Each process has an unique identifier (the `pid`), which we used in the C code above. 
> Since a process is able to [[03 - Process Management#`exec`|launch other processes]], we often represent them as a tree (processes also have a `ppid`, which is the `pid` of its direct parent).

![[process.png]]

This way, the OS also acts as a **referee**, as it is his job to manage protection, isolation and sharing of resources between processes.
The OS is also here to prevent unauthorised access of a process to another process' memory space. If a process tries to access restricted resources, the OS **isolates and terminates it** to prevent further damage.
	*The classic `segmentation fault (core dumped)` that you experience in C when you don't understand how to handle pointers :)*

### Protection

But the OS has to run several processes. A dedicated mechanism that allows to "run and switch processes" exists: **context switching**.
1. **Saving the state** of the currently running process (register values, program counter, etc.).
2. **Loading the state** of the next process to be executed.
3. Scheduling the CPU to **execute the next process**.

This allows multiple processes to share the CPU, creating the illusion that they are running simultaneously.
	*Processes are isolated from each other, and the OS isolates himself from processes. In this context, the OS acts as **a glue** that provides common services, such as networking and storage, to integrate hardware and applications together.*

Furthermore, only the OS should be allowed to perform critical operations (I/O, Memory management...). When a program needs a critical functionality, it will request it via a **system call** (more on that later).


***
## The four fundamentals

### Concurrency

This term refers to a host of problems that arise, and must be addressed, when working on many things at once (concurrently) **in the same program**.
	*This gives room for **parallelism**, as we take advantage of actual hardware parallelism (multicore), and also allows to handle blocking actions (I/O) and simultaneous events.

As explained [[01 - Threads (EN)|here]], the backbone of concurrency is the concept of **thread**, which is simply an execution context in the process.
When executing, a thread takes an entire CPU core when it is **resident** in the processor registers. So, a thread is a **virtual core**.

> [!info] Resident?
> Resident means the registers hold the context (A.K.A. root state) of the thread. 
> In other words, the thread is active and its properties are loaded into memory: its own stack, its own instructions, its own local variables, its **Program Counter (PC)** which indicates the next instruction to execute, and the registers.

If a thread's state is not loaded into the processor (so, not a resident), it is **suspended**.

#### Multiple Processors

Before digging into concurrency, let's see how the OS switches between threads.
The problem is: The OS has to run a higher number of threads than cores available on our CPU. 
Let's assume we only have one core, how do we run all of our threads? Multiplex in time:
![[vcores.png]]
> [!info] 
> Each case is a specific thread. There is a rotation in the flow of execution by the OS so every thread can get the spotlight for a moment. Thread 1 is executed, then it is suspended and thread 2 is executed, and so forth.

If `vCPU1` (thread 1) is resident (actively executed), it is on the real physical core while `vCPU2` and `vCPU3` are saved in their respective chunk of memory called the **Thread Control Block (TCB)**. 
The switch flow is the following:
![[multiplex.png]]
1. `vCPU1` was ran on the physical core by the OS (how? we'll see later), but its alive time ended.
2. We save its PC, Stack pointer ... to `vCPU1`'s **Thread Block Memory** in the TCB
3. Now its `vCPU2`'s turn to be executed. We load `vCPU2`'s TCB (containing PC, SP ...) and jump to PC.
	*We are now back to the instruction we left thread 2 on suspend*

#### Thread Control Block (TCB)

As we explained, when a thread is not running, its dedicated TCB holds:
- Contents of registers
- Program Counter
- Stack Pointer (Points to the thread's dedicated stack held by the process)
- State of the thread (running, ready, waiting, start, done)
- A thread identifier `tid` (similar to a process' `pid`)
- A pointer to the [[ESIPE/Semestre 3/System Programming/01 - Fundamentals/01 - Fundamentals#Scheduling and Process Control Block|Process Control Block (PCB)]] of the process that the thread lives on.
	*This is very similar to the TCB, but for the encapsulating process*

TCBs are stored in the **kernel** (more on that later).

#### Concurrency problems

Our example above was based on an architecture with only one physical core, which is rare. If we can handle more, several threads will run at the same time. Threads in a same process shares the same heap, which causes a lot of issues when they try to access a resource in it at the same time.
	*A dedicated concurrent programming class can be found [[Concurrence|here]]. This goes in depth of the "programming" aspect (how to make programs "thread-safe"), but we will keep talking about concurrency and multithreading here as this class does not cover the OS point of view much.*


### Address Space

The address space is the **set of accessible addresses + state associated with them**.
![[ram.png]]

As we have explained earlier, OS must ensure a security context for each threads.
	*Limit the scope of what a thread can do, the data it can access, prevents a thread launched by user A to access data of a thread launched by user B...*

#### Base and Bound

B&B is a simple memory protection mechanism relying on:
- A **Base Register**, which indicates the starting address of a process's memory
- A **Bound Register**, which indicates the limit of the process' memory region
In other words, the mechanisms ensure that the process only manipulates memory in his  range: `baseRegister ≤ <addr> < boundRegister`

![[b&b.png]]
> [!info]
> Here, the manipulated address is `0x1010`. Since it is in between the boundaries (`0x1000` and `0x1100`), everything is alright.

#### Relocation

The concept is the following:
1. Compiled object files are linked together to create the final executable file we launch. All addresses **in compiled code** assume a start at `0x00000000`.
2. **At runtime**, these addresses are **adjusted by adding the base value of where the process is loaded**.
	*So, if the executed address is `0x0010` and the base range address is `0x1000`, we do a simple padding to relocate the address inside the range: `0x0010 + 0x1000 = 0x1010`*
![[relocation.png]]

#### x86 Segment registers

The x86 architecture uses segment registers (`CS`, `SS`, etc.) to **define the start, length, and access rights of different memory segments**:
- **Data Segment (DS)**: Variables and static data.
- **Code Segment (CS)**: Instructions
	*As explained [[02 - Généralités sur l'assembleur#Registres|here (in French)]], `EIP` points to the next instruction to execute. So, `EIP` starts at `CS`, and goes downwards as instructions are being executed.*

- **Stack Segment (SS)**: Function call stack.
	*`ESP` points to the top of the stack. It starts at `SS` and goes downwards.*

#### Paged Virtual Address Space

> [!question]
> To ensure isolation and prevents direct access to physical addresses, processes operates in a virtual address space. 
> But how is the physical address **translated** into virtual addresses?

We go with a **paging mechanism** that divides memory into equal-sized **pages** in the virtual address space and **frames** in physical memory. Now, those pages can be mapped to any physical frame, which offers great flexibility!
![[page.png]]


### Process (more in depth)

As we know, a program is executed as a process. A process is simply an execution environment with its dedicated address space, encapsulating one or more threads.
	*A program can be on several processes, via `fork` and `exec`, but we will talk about this later on.*
![[multithread.png]]
> [!info] 
> Here we have a visual representation of what we claimed previously: Each thread has its own registers and stack, but the machine code, heap and **file descriptors** are shared among all threads.

Processes guarantee **security and reliability**: a malicious or compromised process can’t read or write other
process’ data, and bugs cannot affect other processes.
This overall concept is a tradeoff: Communication within a process (between threads) is made easier, but communication between processes tends to be more difficult due to the isolation.

But why can't a process leverage I/O instructions or manually change the page table pointer to bypass this security? This is due to **privilege levels**.


### Dual Mode Operation

CPU provides rings that defines privilege modes. Only two of them really matters:
- Ring 0: Kernel mode (or "supervised" mode)
- Ring 3: User mode
More details on rings [[01 - Hardware Virtualization#CPU Virtualization|here]].

#### Syscalls

Several operations are restricted by the OS. If a process wants to have access to them, they must **delegate the action to the OS**; mostly through **system calls**, but also interrupts and exceptions.
	*Those are called "Unprogrammed control transfer" as it doesn't run on direct instructions from the running process, unlike standard instructions.*
![[delegate.png]]

**Syscalls** are like an API: the process requests a system service the OS will execute on his behalf.
	*It's like a function call but outside of the process, so we don't have the address of the system function to call. Instead, this works by invoking the syscall id and placing arguments in registers:*
```asm
mov ebx, 0 ; will contain the exit value
mov eax, 1
int 0x80   ; "int" stands for "interrupt", not "integer"...
```
> [!info]
> This syscall stops the program.

#### Interrupt Vector

Each interrupt has a corresponding **handler**. The CPU uses an **Interrupt Vector Table** (IVT) to locate the address of the appropriate handler based on the interrupt number.

#### Scheduling and Process Control Block

Kernel represents each process as a Process Control Block (PCB).
Similarly to TCBs, a PCB contains information about a process:
- Status (running, ready, blocked, …)
- Register state (when not ready)
- Process ID (PID), User, Executable, Priority, …
- Execution time, …
- Memory space, translation, …

The **Scheduler** maintains a data structure containing all the PCBs, and the scheduler's algorithm (round-robin, priority-based...) picks manages which threads inside those PCBs can run or not.
	*The scheduler is the entity that manages which threads are running or not, as we have described [[ESIPE/Semestre 3/System Programming/01 - Fundamentals/01 - Fundamentals#Multiple Processors|here]] *
```c
if (readyThreads(TCBs)) { // Check if there are threads ready to run
    nextTCB = selectThread(TCBs); // Select the next thread to run
    run(nextTCB);                 // Run the selected thread
} else {
    run_idle_thread();    // No threads are ready, run the idle thread
}
```
