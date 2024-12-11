[[System Programming]]
****
**Table of Contents**
```table-of-contents
```

****
## (Safe) Kernel mode transfers

Syscall table—along with the usage of a separate kernel stack—allows a controlled transfer into kernel.
	*As long as kernel code is reliable, it will securely packs up the user process state and sets it aside. (a malicious process is not supposed to be able to corrupt the kernel)*

As we briefly mentioned, there are three types of kernel mode transfer:
1. **Syscall**: A process requests a system service (e.g., `exit`, `fork`...).
	*It's like calling a function outside of the process. This means that we don't have the address of this function. *
2. **Interrupt**: Independent of the process itself, it's usually an external asynchronous event triggers context switch (e.g., timer, I/O device ...)
3. **Exception**: Any internal synchronous event in process triggers context switch (e.g., violating the memory protection, dividing by 0 ...)

### Syscall

To handle syscalls safely, kernel has to protect itself:
- **Well-defined syscall entry points**: A table maps syscall id to the according handler
- **Atomic switch**: Atomically set to kernel mode at same time as jump to system call code in kernel
- **Defensive copy and validation**: 
	- (Input) All given arguments are copied into the kernel memory and then verified (invalid values, illegal addresses).
	- (Output) Result is copied into user process memory.
> For this last point, this logic is kind of similar to what we did in Java: We **never trust what's coming from the outside** (not `private`), so we always include pre-conditions and defensive copies.

### Interrupt & Exceptions

We handle both the same way:




****
## Process management API

Everything running outside of the kernel is a process. But how do we launch one, stop it, make it launch another process..?

