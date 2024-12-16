[[System Programming]]
****
**Table of Contents**
```table-of-contents
```

****
## Pipes - Concept
*Details: `man 7 pipe`*

Pipe provides an **unidirectional communication channel** from one process to another.
The channel is a **stream of bytes** that maintains the order of bytes (FIFO). 
**Reading is destructive**. In other words, information that has been read disappears from the pipe.
```
|-------------|                        |-------------|
|  process 1  |    |--------------|    |  process 2  |
| (0)     (1) | -> | (1) pipe (0) | -> | (0)     (1) |
|     (2)     |    |--------------|    |     (2)     |
|-------------|                        |-------------|
```
> 0 = `STDIN`, 1 = `STDOUT`, 2 = `STDERR`

We create a pipe like so: `int pipe(int pipefd[2]);`
As we know well for C, pointers passed as parameters are often used as return slots (the function will fill the memory space pointed by it). It is the case here: The function will **fill our array with two file descriptors**:
- `pipefd[0]`: the read end of the pipe (reads from process 1)
- `pipefd[1]`: the write end of the pipe (writes to process 2)

The two `fd` refers to a inode in a special pipe file system. This inode leverages **its own buffer to temporarily store the data** written to the pipe.
	*Default pipe capacity on Linux is 16 pages (=64 KiB on x86):*
```c
int main(void) {
	int p[2];
	try(pipe(p));
	try(fcntl(p[1], F_SETFL, O_NONBLOCK));
	size_t i;
	
	for (i = 0; write(p[1], "#", 1) != -1; i++) {}
	if (errno != EAGAIN) {
		perror("write"); 
		exit(EXIT_FAILURE);
	}
	printf("The pipe capacity is %ld bytes.\n", i);
}
```
```bash
$ ./pipe-capacity
The pipe capacity is 65536 bytes.
```


***
## Read and Write through pipes
*Through `read()` and `write()`*
### Blocking pipes

```
|----------|                        |-------------|
|  writer  |    |--------------|    |    reader   |
|      (1) | -> | (1) pipe (0) | -> | (0)         |
|          |    |--------------|    |             |
|----------|                        |-------------|
```

| Writing                        | Pipe       | Reading            |
| ------------------------------ | ---------- | ------------------ |
|                                | empty pipe | `read()` blocks    |
| no writer                      | empty pipe | `read()` returns 0 |
| `write()` blocks               | full pipe  |                    |
| `write()` raises <br>`SIGPIPE` |            | no reader          |

### Non-blocking pipes

```
|----------|                        |-------------|
|  writer  |    |--------------|    |    reader   |
|      (1) | -> | (1) pipe (0) | -> | (0)         |
|          |    |--------------|    |             |
|----------|                        |-------------|
```
> both `pipefd` are declared with `fcntl(1, F_SETFL, O_NONBLOCK);`

| Writing                           | Pipe       | Reading                          |
| --------------------------------- | ---------- | -------------------------------- |
|                                   | empty pipe | `read()` fails <br>with `EAGAIN` |
| no writer                         | empty pipe | `read()` returns 0               |
| `write()` fails <br>with `EAGAIN` | full pipe  |                                  |
| `write()` raises <br>`SIGPIPE`    |            | no reader                        |

### Close unused pipe ends

```c
int main(void) {
	int p[2]; 
	try(pipe(p));
	
	switch (try(fork())) {
		case 0:               // Child (writer)
			try(close(p[0])); // <-- close reader fd
			try(write(p[1], "123456789", 9));
			printf("Writer done.\n"); 
			break;
		default:              // Parent (reader)
			try(close(p[1])); // <-- close writer fd
			char buf[2] = " ";
			while (try(read(p[0], buf, 1)) > 0)
				try(write(1, buf, 2)
			);
			printf("Reader done.\n"); 
			break;
	}
}
```
```
$ ./good-pipe
1 2 3 4 5 6 7 8 9 Writer done.
Reader done.
```
> Refresher: `fork()` duplicates file descriptors. So, each process should begin by closing the pipe end he doesn't use.

### Deadlocks

Like in concurrent programming, some situations can lead to deadlocks when using pipes.
A first—and silly—example is when doing a process reads on its own pipe:
```c
int c, p[2];
try(pipe(p));
try(read(p[0], &c, 1));
```
```
	   |----------|
/----> | process  | -----\
|	   |----------|      |
|				         |
|				         |
|    |--------------|    |
\--- | (0) pipe (1) | <--/
	 |--------------|
```

A more common deadlock situation is usually between two processes, either because:
- both read from an empty pipe
- both write to a full pipe
```
	     |-----------|
   /---> | process 1 | ----\
   |     |-----------|     |
   |                       |
   |                       ↓
|------|                |------|
| (0)  |                |  (1) |
|      |                |      |
| pipe |                | pipe |
|      |                |      |
|  (1) |                | (0)  |
|------|                |------|
   ↑                        |
   |                        |
   |      |-----------|     |
	\---- | process 2 | <---/
          |-----------|
```

Let's take the following situation:
```c
int main(void) {
	int c, p2c[2], c2p[2];
	try(pipe(p2c)); 
	try(pipe(c2p));
	
	switch (try(fork())) {
		case 0:
		try(close(p2c[1]));
		try(close(c2p[0]));
		try(read(p2c[0], &c, 1));
		printf("Child terminated.\n"); 
		break;
	default:
		try(close(p2c[0]));
		try(close(c2p[1]));
		try(read(c2p[0], &c, 1));
		printf("Parent terminated.\n"); 
		break; 
	}
}
```
```bash
$ ./deadlock-2 &
$ ps -o comm --forest
COMMAND
sh
\_ deadlock-2    # Both processes block, waiting for the other 
| \_ deadlock-2  # to write something.
\_ ps
```

### Synchronise

A good way of using pipes is through a synchronisation logic:
```c
int main(void) {
	int dummy, p[2];
	try(pipe(p));
	
	for (int i = 1; i <= 3; i++)
		switch (try(fork())) {
			case 0: // Child
			try(close(p[0]));
			sleep(i);
			printf("Child %d done\n", i);
			exit(EXIT_SUCCESS);
		default: 
			break; // Parent
	}
	
	try(close(p[1]));
	try(read(p[0], &dummy, 1));
	printf("Parent continues\n"); 
}
```
```bash
$ ./pipe-sync
Child 1 done
Child 2 done
Child 3 done
Parent continues
```
> Here, the parent blocks until all its children have closed the pipe’s writing end.

### Named Pipes

What we have seen so far were **anonymous pipes**. However, those does not allow processes without parent-child relationship to be connected.

We will simply use **named pipes (FIFOs)** instead. As the name suggests, they are like ordinary pipes, except they have a name within the file system.

We declare one with: `int mkfifo(const char *pathname, mode_t mode);`
	where `pathname` is its name, and `mode` the permissions.
The named pipe:
- **Can be opened** like any ordinary file
- Has to be **opened at both ends simultaneously** (i.e., read & write) to do any I/O
- By default, `open()` for reading blocks until some other process calls `open()` for writing, and vice versa. 
	*This can be disabled with `O_NONBLOCK`
```bash
# Shell 1                   # Shell 2
$ mkfifo fifo               $ cat < fifo
$ echo "Hi!" > fifo         Hi!
                            $
```
> `echo "Hi!" > fifo` blocks until Shell 2 opens `fifo`

### Branching

Shell pipelines are linear, but we can introduce branches by combining FIFOs with the `tee` command.
	*The `tee` command copies its standard input to both its standard output and any files it gets as arguments.*

```bash
$ mkfifo fifo
$ printf "BB\nCC\nAA\n" | tee fifo | sort &
$ wc -l < fifo
AA
BB
CC
3
```
```
|----------|     |------|     |-------|     |------|     |--------|
|  printf  | --> | pipe | --> |  tee  | --> | pipe | --> |  sort  |
|----------|     |------|     |-------|     |------|     |--------|
                                  |
	                              |         |------|     |--------|
	                              \-------> | FIFO | --> |   wc   |
	                                        |------|     |--------|
```


***
## (POSIX) Semaphores

A semaphore is an OS-managed object holding a **non-negative integer value** defining its availability, and a **waiting queue containing pointers towards threads awaiting to access the semaphore**. 
	*Its a primitive FIFO synchronisation mechanism—initially based on Dijkstra. As long as the integer value hasn't reached 0, it can be used atomically by a thread.*

Once declared, only two operations are permitted:
- `wait()`: Atomic operation that waits for semaphore to become positive, then decrements it by 1
	*So, this is a blocking action. Sometimes also called `P()`, which stands for **P**roberen te verlage (**TRY** to decrease)*
- `post()`: Atomic operation that increments the semaphore by 1, waking up a waiting P, if any.
	*Equivalent to a `signal()`. Sometimes also called `V()`, which stands for **V**erhogen (increase)*
> This means that we can not directly read/write the integer value after initialisation, it is **opaque**. Furthermore, those two methods must necessarily be atomic (so if several `P()` occurs at the same time, the integer value cannot drop bellow 0).

Try to represent it like a traffic light turning red when the integer reaches 0, thus blocking all access via `P()` until someone uses `V()` to come back:
![[semaphore.png]]


```c
#include <semaphore.h>
#include <fcntl.h>     // handy for oflag

/*
* Initializes or opens a named semaphore.
* The name is given by <name>, which must start with a '/'.
* 
* <oflag> determines whether to create a new semaphore or open an existing one.
* - O_CREAT: Create a new semaphore if it doesn’t exist.
* - O_EXCL: Ensure a new semaphore is created (fails if it already exists).
* 
* <mode> defines permissions for the semaphore.
* (Used only if O_CREAT is specified).
* Logic is similar to file permissions (e.g., 0666 for read/write access).
* 
* <value> is the initial positive integer value for the semaphore 
* (Used only if O_CREAT is specified).
* 
* Returns a pointer to the semaphore or NULL on failure.
*/
sem_t *sem_open(const char *name, int oflag, mode_t mode, unsigned int value);

/*
* Removes a named semaphore.
* The name is given by <name>, which must start with a '/'.
* 
* - The semaphore will no longer be accessible by this name.
* - If processes still have the semaphore open, it will remain available 
*   until all references are closed (e.g., with sem_close()).
* 
* Returns:
* - 0 on success.
* - -1 on failure (e.g., if the semaphore does not exist or permission is
*    denied).
* 
* Typical use:
* - Ensure proper cleanup of named semaphores when they are no longer needed.
*/
int sem_unlink(const char *name);

/*
* Initializes an anonymous semaphore.
* The semaphore is given by <sem>.
* 
* <pshared>:
* - If 0, the semaphore is shared only between threads of the same process.
* - If non-zero, the semaphore is shared between processes (requires shared
*   memory segment).
* 
* <value>: The initial positive integer threshold for the semaphore.
*/
int sem_init(sem_t *sem, int pshared, unsigned int value);

/*
* Destroys an anonymous semaphore.
* The semaphore is given by <sem>.
* 
* - Frees any resources allocated for the semaphore.
* - Can only be called when no threads or processes are waiting on the semaphore.
*/
int sem_destroy(sem_t *sem);



/*
* Decrements (locks) the semaphore.
* The semaphore is given by <sem>.
* 
* - If the semaphore's value is greater than zero, decrements it and continues.
* - If the semaphore's value is zero, blocks until it becomes greater than zero.
* - Ensures access control by blocking threads/processes until resources are
*   available.
*/
int sem_wait(sem_t *sem);

/*
* Increments (unlocks) the semaphore.
* The semaphore is given by <sem>.
* 
* - Increments the semaphore's value by 1.
* - If there are threads or processes blocked on sem_wait(), wakes one of them.
* - Used to signal that a resource has been released and is available.
*/
int sem_post(sem_t *sem);
```

### Use

Mostly two ways of using a semaphore:
- As a MutEx (initial value = 1)
```c
sem_t mutex;

void* critical_section(void* arg) {
    sem_wait(&mutex); // Lock
    printf("Thread %d in critical section\n", *(int*)arg);
    sem_post(&mutex); // Unlock
    // ...
}
```

- As a scheduling constraint (initial value = 0) to control order of thread execution.
	*Allow thread 1 to wait for a signal from thread 2. thread 2 schedules thread 1 when a given event occurs.*
Example: Manual implementation of `pthread_join` which must wait for thread to terminate:
```c
sem_t done;

void* thread1_code(void* arg) {
    printf("Thread 1 waiting for Thread 2 to finish...\n");
    sem_wait(&done); // Wait signal
    printf("Thread 1 resuming work\n");
    return NULL;
}

void* thread2_code(void* arg) {
	// ...
    printf("Thread 2 finished, signaling Thread 1\n");
    sem_post(&done); // Sends signal to thread1
    return NULL;
}
```


***
## Monitors

Improvement of semaphores: We use a lock for mutual exclusion, and a **condition** variable for scheduling constraints.

Monitors represent the synchronisation logic of the program
- We wait if necessary
- We signal when change something so any waiting threads can proceed

This is something we know well, it is what we have been doing in Java for the [[04 - Signals|concurrent programming class]].
	*Java provides this feature natively... A monitor is only a **paradigm** of concurrent programming.*

We use those methods to simulate it in C:
```c
int pthread_cond_init(pthread_cond_t *cond, const pthread_mutexattr_t *attr);

// lock.wait()
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);

// lock.notify()
int pthread_cond_signal(pthread_cond_t *cond);

// lock.notifyAll()
int pthread_cond_broadcast(pthread_cond_t *cond);
```

So, a **Mesa Monitor** program will have this structure we know well:
```c
lock
while (need to wait) { // Again, we always put a wait in a while loop
	condvar.wait();    // in case of spurious wakeup
}
unlock

lock
condvar.signal();
unlock
```
