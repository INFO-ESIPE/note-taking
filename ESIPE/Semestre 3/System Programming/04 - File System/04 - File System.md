[[System Programming]]
****
**Table of Contents**
```table-of-contents
```

****
## File System
*For Linux*

The file system is here to create structure around raw data:
- directory hierarchy with file names
- metadata (file size, modification date, owner, ...)

We can represent I/O operations like this:
![[iceberg io.png]]
> [!Note]
> In this lesson, we will focus on both High-level and Low-level I/O, along with getting familiar with the concept of
> File System (we will take a deeper dive in how file system works in the future).
> 	*Other concepts, such as Drivers or Sockets, will be studied in another lesson*

### File System Layout

The physical disk itself only contains partitions (logical slicing) along with a **Master boot record (MBR)** placed at the very beginning of the disk.
The MBR is the part that actually **holds the information on how the disc's blocks are divided into partitions**.

Each partition has its own file system. Let's take a look at what composes a basic file system:
- **boot block**: code used to load the OS of that partition.
- **super block**: state of the file system (partition size, inode number of the root, ...)
- **inode table**: linear array of **inodes**.
- **data blocks**: actual contents of files and directories.
![[filesystem.png]]

### Inode

Each inode **represents a file**. It contains:
- all the metadata (except the file name, we'll see why [[03 - Process Management#Links|later]])
- references to the file’s data blocks (actual content of the file)

Reference to data blocks can be direct or indirect. Since blocks are of a fixed size, it is sometimes mandatory to **maintains a multi-level tree structure** to find storage blocks for large files.
![[inodes.png]]
> Mind the tree-like architecture allowed by indirect pointers

![[pointer-direct.png]]
![[pointer-indirect.png]]

### File

A POSIX file is a named entity containing a **sequence of bytes** (text, binary, serialised objects...) along with **metadata** describing information about the file (Size, Modification Time, Owner, Security info, Access control).

### Directories

A directory is a special file whose data consists of a sequence of entries.
Each entry **associates a file name with an inode number**.
	*At least two entries (required for relative paths): `.` (the current directory) and `..` (the parent directory)*
![[directory.png]]

### Links

There are two types of links:
1. **Hard Link**:
	- Inodes count their number of hard links.
	- Multiple hard links allowed on regular files.
	- Only 1 on directories (no cycles).
2. **Symlink**:
	- Under the hood, this is **just a file containing a path**.
	- Breaks if the target disappears.
	- Cycles are allowed (unlike hard links).
![[links.png]]

> [!info] Everything is a file (stream of bytes)
> Everything we manipulate on the OS is represented as a file. This explains why there are so many "types" of files on Linux:
> 	Regular files, Directories, Symlinks, IPCs (pipes, network sockets), Hardware devices (keyboards, printers, disks...), and Pseudo-devices (`/dev/null`, `/dev/random`, `/dev/zero`...)
>
> This approach allows all of those elements to share the same API. All those objects will be represented by **file descriptors**.


****
## File API

### High-Level
*High-level functions comes from `stdio.h`, and their entry is in `man 3`.

Those operates on “streams” – unformatted sequences of bytes (wither text or binary
data), with a position.
	*Usually, they start with the `f` character*.

```c
#include <stdio.h>

/*
* stream-oriented: unformatted sequences of bytes (wither text or binary
*                  data), with a position.
* Permissions are passed in the "mode" parameter:
* - r   Open existing file for reading
* - w   Open for writing; created if does not exist
* - a   Open for appending; created if does not exist
* - r+  Open existing file for reading & writing.
* - w+  Open for reading & writing; truncated to zero if exists, create otherwise
* - a+  Open for reading & writing. Created if does not exist. Read from
*       beginning, write as append.
*/
FILE *fopen( const char *filename, const char *mode);
int fclose(FILE *fp);

// character oriented
int fputc(int c, FILE *fp);              // return char or EOF on error
int fputs(const char *s, FILE *fp);      // return > 0 or EOF
int fgetc(FILE * fp);
char *fgets(char *buf, int n, FILE *fp);

// block oriented
size_t fread(void *ptr, size_t size_of_elements, 
			 size_t number_of_elements, FILE *a_file);
size_t fwrite(const void *ptr, size_t size_of_elements, 
			  size_t number_of_elements, FILE *a_file);

// formatted
int fprintf(FILE *restrict stream, const char *restrict format, ...);
int fscanf(FILE *restrict stream, const char *restrict format, ...);
```
> [!caution]
> An opened file should ALWAYS be closed afterwards.

Here is an example of a char-by-char I/O that opens and closes files correctly:
```c
FILE* safe_fopen(const char* filename, const char* mode) {
    FILE* file = fopen(filename, mode);
    if (file == NULL) {
        perror("Failed to open file");
        exit(EXIT_FAILURE);
    }
    return file;
}

int main(void) {
	FILE* input = safe_fopen("in.txt", "r");
    FILE* output = safe_fopen("out.txt", "w");

	int c = fgetc(input);
	while (c != EOF) {
		fputc(output, c);
		c = fgetc(input);
	}

	fclose(input);
	fclose(output);
}
```

And here is an example of block-by-block I/O that opens and closes files correctly:
```c
#define BUFFER_SIZE 1024

int main(void) {
	FILE* input = safe_fopen("in.txt", "r");
    FILE* output = safe_fopen("out.txt", "w");
    
	char buffer[BUFFER_SIZE];
	size_t length = fread(buffer, sizeof(char), BUFFER_SIZE, input);
	while (length > 0) { // while there is still characters to read
		fwrite(buffer, sizeof(char), length, output);
		length = fread(buffer, sizeof(char), BUFFER_SIZE, input);
	}
	
	fclose(input);
	fclose(output);
}
```


### Low-Level
*Low-level functions comes from `stdlib.h`, and have a dedicated entry in `man 2`.*

Most low-level functions represent files using a **file descriptor**, which is an integer value associated with a file in the current process's **file descriptor table**. This file descriptor points to an entry in the system-wide **open-file table**, which in turn references the file's **inode**.
![[filedescriptor.png]]
> [!info] 
> Each file is uniquely identified by its inode within the filesystem, and file descriptors provide a convenient way for processes to interact with these inodes indirectly.

Let's see an example of this phenomenon by looking at how the major low-level file API function behaves: `int open(const char *pathname, int flags)`:
1. Find the inode of the requested file.
2. Load the inode into memory (if it is not already there).
3. Check the file’s permissions to see if it can be accessed.
	*If the permissions of an open file are revoked, the file can still be accessed!*
4. Allocate a new entry in the the system-wide table of open files.
5. Position the file offset (0 by default, or `EOF` in we chose an "append mode").
6. Allocate a new entry in the file descriptor table of the current process.
7. Return an integer value which corresponds to the obtained file descriptor

Now that we are familiar with the concept of file descriptor, let's take a look at the most common low-level functions:
```c
/*
* Opens the file specified by pathname and returns a file descriptor 
* referring to it. The file remains opened until closed with close() 
* or the process terminates.
* 
* "flags" must include one of the following permission:
* - O_RDONLY: read-only
* - O_WRONLY: write-only
* - O_RDWR: read and write
* It can also include "bitwise-or’d" optional flags:
* - O_CREAT: Create the file if it does not exist (see next slide).
* - O_APPEND: The file is opened in append mode.
* - O_TRUNC: Truncate the file to length 0 (if it exists).
* ...
*/
int open(const char *pathname, int flags);

/*
* A variant of the function above. Opens the file specified by pathname and
* creates it if it doesn't exist.
*
* "mode" indicates the file mode bits applied when a new file is created
* (UNIX permissions):
* - 0600: gives the user read and write permissions.
* - S_IRUSR|S_IWUSR: does the same
*/
int open(const char *pathname, int flags, mode_t mode);

// Same as tbe first open() with flags equal to O_CREAT|O_WRONLY|O_TRUNC.
int creat(const char *pathname, mode_t mode);

/*
* Closes the file descriptor fd so that it no longer refers to any file.
* 
* If fd was the last file descriptor referring to the underlying open file,
* the associated resources are freed.
*/
int close(int fd);

/*
* Deletes pathname from the filesystem 
* and decrements the link counter of the associated inode.
* If pathname was the last link to a file, the file is deleted.
*/
int unlink(const char *pathname);

/*
* Attempts to read up to "count" bytes from file descriptor fd 
* into the buffer starting at "buf".

* It can read fewer than "count" if the process was interrupted by a signal, or
* if EOF is reached.
*/
ssize_t read(int fd, void *buf, size_t count);

/*
* Attempts to write up to "count" bytes to file descriptor fd 
* from the buffer starting at "buf".

* It can write fewer than "count" if the process was interrupted by a signal, or
* if there is not enough space on the underlying physical medium.
*/
ssize_t write(int fd, const void *buf, size_t count);
```
> [!caution] 
> There is no re-check of permissions after an `open()`! Which means, if permission on the file changes after a process acquired the file descriptor, the process can still act on the file like nothing happened.  

### Differences between Low-level and High-Level API

Alright, we now see that there are two ways of doing it, but what are the fundamental differences between high and low level operations?

For High-level API, Streams are **(line-)buffered in user memory**:
```c
printf("Beginning of line ");
sleep(10);
printf("and end of line\n");
```
> [!info]
> Everything is printed at once. In other words, we have to wait 10 seconds for something to print (because the first `printf` doesn't end with `\n`).

On the other hand, Low-level API operations are visible immediately:
```c
write(STDOUT_FILENO, "Beginning of line ", 18);
sleep(10);
write("and end of line \n", 16);
```
> [!info]
> "Beginning of line " printed 10 seconds earlier than “and end of line”.


### Standard Streams

As we saw above, functions opening a file will either return:
-  A `FILE*` pointer: `fopen` in `<stdio.h>` 
- A file descriptor: `open`/`creat` in `<stdlib.h>`

Three predefined streams are opened implicitly when the program is executed:

|                 | File descriptor    | FILE* pointer |
| --------------- | ------------------ | ------------- |
| Standard input  | 0, `STDIN_FILENO`  | `stdin`       |
| Standard output | 1, `STDOUT_FILENO` | `stdout`      |
| Standard error  | 2, `STDERR_FILENO` | `stderr`      |
> Those macros are respectively given in `unistd.h` and `stdio.h`, as explained above.


### Reposition file offset

`off_t lseek(int fd, off_t offset, int whence);` repositions the file offset of the open file associated with `fd` to the position `whence + offset` (bytes)”.
- `offset` can be negative
- `whence` can be:
	- `SEEK_SET` - start of the file
	- `SEEK_CUR` - current position in the file
	- `SEEK_END` - end of the file

![[lseek.png]]
> [!caution]
> `lseek()` can go beyond the end of a file. If data is later written at the new position, reading in the resulting gap returns null bytes until real data is written into the gap.

Some devices (pipes, terminals) does not support `lseek()`:
```c
int main(void) {
	if (lseek(STDIN_FILENO, 0, SEEK_CUR) == -1) {
		printf("STDIN not seekable\n");
	}
	else {
		printf("STDIN seekable\n");
	}
}
```
```bash
$ ./seekable            # terminal
STDIN not seekable

$ ./seekable <file      # regular file
STDIN seekable

$ cat file | ./seekable # pipe
STDIN not seekable
```

`lseek` may introduce **file holes** if used in a certain way. If a file contains some, its physical size (on disk) can be much smaller than its logical size:
```c
// Creates a big hole
int main(void) {
	int fd = try(creat("myfile", 0644));
	try(lseek(fd, 1000000, SEEK_SET));
	try(write(fd, "Hello\n", 6));
	try(close(fd));
}
```
```bash
$ ./big-hole
$ od -cAd myfile
0000000 \0 \0 \0 \0 \0 \0 \0 \0 \0 \0 \0 \0 \0 \0 \0 \0
*
1000000 H e l l o \n
1000006

$ cat myfile > myfile-copy
$ ls -ls myfile*
    4 -rw-r--r-- 1 alice alice     1000006 Nov 9 10:06 myfile
  980 -rw-r--r-- 1 alice alice     1000006 Nov 9 10:07 myfile-copy
# ↑ physical size in 1kib blocks   ↑ logical size in bytes
```


### `dup2`

`int dup2(int oldfd, int newfd);` creates a copy of the file descriptor `oldfd` and uses `newfd` for the new file descriptor. 
Both the new and old file descriptors now refer to the same open file description:
- They share the same file offset and file status flags.
- Both have to be closed to close the underlying file.
	*If `newfd` was previously open, it is silently closed before being reused.*

```c
int main(void) {
	int fd = try(creat("myfile", 0644));
	try(dup2(fd, STDOUT_FILENO));
	try(close(fd));
	try(write(STDOUT_FILENO, "Hello\n", 6));
}
```
```bash
$ ./modern
$ cat myfile
Hello
```
> [!caution] 
> As you may have guessed, the existence of `dup2` implies the existence of `dup`, but the later is deprecated and shouldn't be used (that's why there is another one lol).
