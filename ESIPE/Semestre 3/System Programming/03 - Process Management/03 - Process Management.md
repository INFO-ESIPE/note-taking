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


****
## Process management API
*For more details on each function, use `man 2 <function>`*

Everything running outside of the kernel is a process. But how do we launch one, stop it, make it launch another process..?

> [!question]
> If processes are launched by other processes, how is the first process created ?*

The first process (called `init`) is started by the kernel. After that, any process is created by another process.

### Convenience: the `try()` macro

Most syscalls must be checked for error values (usually -1). This is pretty redundant and not useful to the class, so we will—sometimes—use the following macro which was written by one of my teacher:
```c
#ifndef TRY_H
#define TRY_H

#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define try(...) TRY(, ##__VA_ARGS__, -1)

#define TRY(_, EXPR, ERRVAL, ...)                                     \
  ({                                                                  \
    typeof(EXPR) _retval_ = (EXPR);                                   \
    if (_retval_ == ERRVAL) {                                         \
      fprintf(stderr,                                                 \
              "\033[1m%s:%d:\033[0m "                                 \
              "Call \033[1;36m%s\033[0m failed:\n"                    \
              "\033[1;31m%s\033[0m\n",                                \
              __FILE__, __LINE__, #EXPR, strerror(errno));            \
      _exit(EXIT_FAILURE);                                            \
    }                                                                 \
    _retval_;                                                         \
  })                                                                  \

#endif /* TRY_H */
```

We use it around a function that might fail like so:
```c
try(execvp(argv[1], argv + 1));
```

### `exit`

Allows to terminate a process:
```c
int main(int argc, char *argv[]) {
	printf("My pid: %d\n", getpid());
	exit(0);
}
```

You might be wondering: How is it different from using `return 0;` ?
Well, `return` is the generic keyword to indicate we exit a function. If we use `return` in the main, the OS will call `exit` for us!

### `fork`

Allows to run a copy of the current process.
	*This copy will be another process, direct child of the process which executed `fork`.
	Only has one thread on startup.*
Since we return a copy, it means that **both processes will execute the same instructions starting from the `fork`**.
	*The only difference is the child will have its own pid (since its a new process). Child inherits copies of the parent's open file descriptors.*

In general this behaviour is not exactly what we want, we need **a way to separate the actions of the parent and the child** (so each execute a different part of our code).
We can fix this issue with `fork`'s return value:
- 0 is returned for the child process
- child's pid (> 0) is returned to the parent
- -1 is returned if an error occurred

Now that we know what it returns, here is the correct way to use a fork:
```c
int main(int argc, char *argv[]) {
	pid_t cpid, mypid;
	switch (cpid = fork()) {
		case -1:                // Code executed by parent if fork failed
			perror("fork");
			exit(EXIT_FAILURE);
		case 0:                 // Code executed by the child process
			mypid = getpid();
			printf("[%d] child\n", mypid);
			break;
		default:                // Code executed by parent process
			mypid = getpid();
			printf("[%d] parent of [%d]\n", mypid, cpid);
			break;
	}
}
```
> An `if-elseif-else` can be used as well but its way less elegant and comprehensible. `default` comes handy in this situation as **we cannot predict the child's pid**, but since it will necessarily be a value greater than 0, it will fall in the `default`.

> [!important]
> The parent may run before or after the child, its the scheduler who decides...
> In our example above, we can't know for sure which thread (child or parent process) will print first.

### `wait`

`pid_t wait(int *wstatus);` - Waits (blocking action) for a child of the calling process to die, and reports status information.
	*Similar to `thread.join()` but for an entire process*
```c
int status;
pid_t cpid, tcpid;

switch (cpid = fork()) {
	case -1:
		perror("fork");
		exit(EXIT_FAILURE);
	case 0:  // Child
		mypid = getpid();
		printf("[%d] child\n", mypid);
		exit(42);
	default: // Parent
		mypid = getpid();
		printf("[%d] parent of [%d]\n", mypid, cpid);
		tcpid = wait(&status); // Parent waits for child death before proceeding
		printf("[%d] bye %d(%d)\n", mypid, tcpid, status);
}
```

`wstatus` can point to an integer in which information about the child gets stored.
The integer itself can be inspected with:
- `WIFEXITED()`: Did the child terminate voluntarily (e.g., by calling exit())?
- `WEXITSTATUS()`: If so, what was the child’s exit status (least significant 8 bits)?
```C
int main(void) {
	switch (try(fork())) {
		case -1:
			perror("fork");
			exit(EXIT_FAILURE);
		case 0: // Child
			printf("Child: my PID is %ld\n", (long) getpid());
			exit(42);
		default; // Parent
			int wstatus;
			pid_t pid = try(wait(&wstatus));
			if (WIFEXITED(wstatus))
				printf("child %ld exited with status %d\n", 
						(long) pid,
						WEXITSTATUS(wstatus)
				);
			}
			break;
}
```

A more granular version exists:
`pid_t waitpid(pid_t pid, int *wstatus, int options);`
- `pid` specifies the PID of the child to wait for (or a set of possible children if pid ≤ 0).
- `options` can include the flags:
	- `WNOHANG` to avoid blocking if there is currently no status to report for any child,
	- `WUNTRACED` to also report status for children that have been stopped,
	- `WCONTINUED` to also report status for (stopped) children that have been resumed.

### `exec`

Changes the program being run by the current process.
```c
int main(void) {
	char *args[] = {“ls”, “-l”, NULL};
	execv("/bin/ls", argv);

	// execv doesn't return. If we go beyound the exec instruction, it means that
	// it failed. We must handle it
	perror(“execv”);
	exit(EXIT_FAILURE);
}
```

In fact `exec` is a family, where each function expects specific parameters:
- `int execv(const char *path, char *const argv[]);` - No explicit environment array. Uses the caller’s environment variables instead.
- `int execvp(const char *file, char *const argv[]);` - If file does not contain a /, the file is sought in the directories given in `$PATH`.
- `int execl(const char *path, const char *arg0, ... /* (char *) NULL */);` -Receives a list of arguments `arg0`, `arg1`, . . . , `argn` instead of an `argv` vector.
- A few others: `execvpe()`, `execlp()`, `execle()`

> [!tip]
> You don't really need to learn those by heart, here is a mnemonic:
> `v`: vector, `l`: list, `p`: path, `e`: environment

### fork-exec pattern

A classic `fork–exec` flow can be used:
1. The parent process forks off a child, and then blocks on a wait() call.
2. The child calls (a version of) `exec*()` and thus becomes the new program to run.
3. Once the child terminates, the parent resumes because its `wait()` call returns.
> This is how shell works under the hood :)
![[exec.png]]

**Example:**
```c
int main(int argc, char **argv) {
	if (argc < 2) {
		fprintf(stderr, "Wrong usage\n");
		exit(EXIT_FAILURE);
	}
	
	printf("--> Running `%s`\n", argv[1]);
	switch (try(fork())) {
		case 0:  // Child
			try(execvp(argv[1], argv + 1));
		default: // Parent
			int wstatus;
			try(wait(&wstatus));
			if (WIFEXITED(wstatus))
				printf("--> Returned %d\n", WEXITSTATUS(wstatus));
			}
			break;
}
```

### `kill`

Send an kill signal (interrupt notification) to another process.
```c
#include <stdlib.h>
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <signal.h>

void signal_callback_handler(int signum) {
	printf(“Caught signal!\n”);
	exit(EXIT_FAILURE);
}

int main() {
	struct sigaction sa;
	sa.sa_flags = 0;
	sigemptyset(&sa.sa_mask);
	sa.sa_handler = signal_callback_handler;
	sigaction(SIGINT, &sa, NULL);
	while (1) {} // infinite loop
}
```
> This code can **intercept** a `SIGINT` signal (via a `sigaction`), and deal with it accordingly. If the code doesn't include a handler like above, the process will be terminated.  

Plenty of signals exists:
- `SIGINT` - ctrl-C
- `SIGTERM` - default for kill shell command
- `SIGSTOP` - ctrl-Z (default action: stop process)
*ctrl-D sends an End of file (`EOF`), which isn't the same as sending a kill signal.*


