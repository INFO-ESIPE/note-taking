[[System Programming]]
****
**Table of Contents**
```table-of-contents
```

****
## Concept
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
