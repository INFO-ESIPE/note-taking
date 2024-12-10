[[System Programming]]
****
**Note:** There is already a concurrent programming class detailing how to use threads in a correct way (in Java). We will simply take a look at how it behaves at OS-level by using the dedicated syscalls in C.
****
**Table of Contents**
```table-of-contents
```

****
## Refresher

We are now aware that processes are isolated from each other, that the kernel can switch among user processes, but we still don't know which process to run, nor how do we get the stack and heap to the kernel.

As explained in the previous lesson, the kernel represents each process by a PCB, and the kernel's scheduler maintains a data structure containing those PCBs (allowing it to give CPU time to processes). 
![[pcb.png]]
> PCB

Threads are a mechanism for concurrency (overlapping execution), but also for parallel (simultaneous execution).
- Multiprocessing: Multiple CPUs (cores)
- Multiprogramming: Multiple jobs/processes
- Multithreading: Multiple threads/processes
The scheduler runs threads at its leisure, in any order and interleaving;

A thread has three states:
- RUNNING: running
- READY: eligible to run, but not currently running
- BLOCKED: ineligible to run


****
## `pthread`
*We know how to do it in Java, now let's look at a deeper level*

OS provides an API for threads: `pthreads` (POSIX threads):
```c
// thread is created executing <start_routine> with <arg> as its sole argument
// <attr> will contain infos such as the scheduling policy or the stack size
int pthread_create(pthread_t *thread, 
				   const pthread_attr_t *attr, 
				   void *(*start_routine)(void*), 
				   void *arg);

// Terminates the thread executing this instruction
// <value_ptr> becomes available for any successful join
void pthread_exit(void *value_ptr);

// suspends execution of the calling thread until the target thread terminates
// On return with a non-NULL <value_ptr> the value passed to pthread_exit()
// by the terminating thread is made available in the location referenced by
// <value_ptr>
int pthread_join(pthread_t thread, void **value_ptr);
```


****
## Interleaving and nondeterminism

If we have two threads, we have:
- Two sets of registers
- Two sets of stacks
![[stacks.png]]
> But how do we position stacks, how do we chose their size, and how to prevent violations if a thread attempts to access another thread's stack (like a process violating his address area) ?

Threads will execute at variable speed and in any order, so our task—as developers—is to design programs that can work with any schedule (thread-safe code).
![[scenarios.png]]
> As we can see here, threads are **non-deterministic** (can run and be suspended at any time by the scheduler).
> Our program must work properly in any scenario, and avoid **race conditions**!

