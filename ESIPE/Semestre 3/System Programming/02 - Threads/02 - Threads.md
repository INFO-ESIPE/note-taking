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

Thread shares:
- Heap
- Global variables
- The kernel context (file descriptors, streams, signals, etc.)


****
## Interleaving and nondeterminism

If we have two threads, we have:
- Two sets of registers
- Two sets of stacks
![[stacks.png]]
> But how do we position stacks, how do we chose their size, and how to prevent violations if a thread attempts to access another thread's stack (like a process violating his address area) ?

Threads will execute at variable speed and in any order, so our task—as developers—is to design programs that can work with any schedule (thread-safe code).
![[scenarios.png]]
> [!info]
> As we can see here, threads are **non-deterministic** (can run and be suspended at any time by the scheduler).
> Our program must work properly in any scenario, and avoid **race conditions**!

As explained in the [[Concurrence|concurrent programming class]], several mechanisms are used to achieve this:
- **Synchronisation**: Coordination among threads, usually regarding shared data
- **Mutual Exclusion (MutEx)**: Ensuring only one thread does a particular thing at a time (one thread excludes the others)
- **Critical Section**: Code exactly one thread can execute at once
- **Lock**: An object only one thread can hold at a time
	*The object providing mutual exclusion*


****
## `pthread`
*We know how to do it in Java, now let's look at a deeper level*

OS provides an API for threads: `pthreads` (POSIX threads):
```c
// Thread is created executing <start_routine> with <arg> as its sole argument.
// <attr> will contain infos such as the scheduling policy or the stack size
int pthread_create(pthread_t *thread, 
				   const pthread_attr_t *attr, 
				   void *(*start_routine)(void*), 
				   void *arg);

// Terminates the thread executing this instruction 
// with the return value <retval>
void pthread_exit(void* retval);

// Suspends execution of the calling thread until thread <tid> finishes.
// (Non-NULL) return value of targeted thread is stored in <retval>.
int pthread_join(pthread_t tid, void **retval);

// Initializes mutex pointed to by <mutex> with attributes specified in <attr>
// If <attr> is NULL, default attributes are used
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);

// Locks (acquire) the mutex pointed to by <mutex>
// If the mutex is already locked by another thread, calling thread will block
int pthread_mutex_lock(pthread_mutex_t *mutex);

// Unlocks (release) the mutex pointed to by <mutex>
// A blocked thread can now acquire the lock
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

**Example:**
```c
int common = 162;
pthread_mutex_t common_lock = PTHREAD_MUTEX_INITIALIZER;

void *fun(void *tid) {
	long id = (long)threadid;
	pthread_mutex_lock(&common_lock);   // critical section
	int my_common = common++;           // critical section
	pthread_mutex_unlock(&common_lock); // critical section

	printf("Thread #%lx stack: %lx common: %lx (%d)\n",
			(unsigned long) &id,
			(unsigned long) &common,
			my_common);

	pthread_exit(NULL);
}
```


### Mutex
*For generic details, see [[03 - Synchronize and Thread-Safe (EN)#Mutex|this class]].*

They are represented by the type `pthread_mutex_t`.
```c
// Initialise - Two methods:
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;       // 1
int pthread_mutex_init(ptread_mutex_t *m, 
					   const pthread_mutexattr_t *attr); // 2

// Usage
int pthread_mutex_lock(pthread_mutex_t *mutex));
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);

// Termination
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```
