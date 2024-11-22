[[Architecture]]
****
**Table of Contents**
```table-of-contents
```

****
## GNU Development Tools

For debug/RE purposes, we will use a debugger called GDB. Debuggers are used by programmers to step through a compiled programs, take a close look at its memory and the way he uses registers.

To make the introduction to GDB lighter, let's begin by using **objdump**, which is a simpler tool that allows to examine binary files.

We have a simple C program that we turned into an object file.
```c
#include <stdio.h>
int main() {
	int i;
	for(i = 0; i < 10; i++) {
		puts("Hello, world!\n");
	}
	
	return 0;
}
```

We analyse it on objdump like so:
`objdump -D program.out | grep -A20 main.:`
	*We display the first 20 lines of the main() function. objdump returns a heavy ton of information sometimes, we need to filter the output.*

And obtain this:
```asm
08048374 <main>:
8048374: 55 push %ebp
8048375: 89 e5 mov %esp,%ebp
8048377: 83 ec 08 sub $0x8,%esp
804837a: 83 e4 f0 and $0xfffffff0,%esp
804837d: b8 00 00 00 00 mov $0x0,%eax
8048382: 29 c4 sub %eax,%esp
8048384: c7 45 fc 00 00 00 00 movl $0x0,0xfffffffc(%ebp)
804838b: 83 7d fc 09 cmpl $0x9,0xfffffffc(%ebp)
804838f: 7e 02 jle 8048393 <main+0x1f>
8048391: eb 13 jmp 80483a6 <main+0x32>
8048393: c7 04 24 84 84 04 08 movl $0x8048484,(%esp)
804839a: e8 01 ff ff ff call 80482a0 <printf@plt>
804839f: 8d 45 fc lea 0xfffffffc(%ebp),%eax
80483a2: ff 00 incl (%eax)
80483a4: eb e5 jmp 804838b <main+0x17>
80483a6: c9 leave
80483a7: c3 ret
80483a8: 90 nop
80483a9: 90 nop
80483aa: 90 nop
```

The address corresponding to each instruction is displayed on the left (hex format).
The middle part contains the machine language instructions for the x86 processor (hex format).
	*This is the binary code the CPU understands and executes, but we represent it in hex as it it way more readable*
The right part contains the assembly instructions.

In this example, the instructions are in AT&T syntax (which is slightly different from the Intel one we are used to in this class, notice the usage of `%` and `$`).
This format is used by default. If we don't like it (and i don't) we can force the usage of Intel syntax:
`objdump -M intel -D program.out | grep -A20 main.:`


Now that we are familiar with the basic look of objdump, we can get into GDB.


****
## GDB

We feed gdb with a program like so:
`gdb -q ./program.out`

gdb has a config file located in `$HOME/.gdbinit`. We will add two lines to it:
```bash
set debuginfod enabled on # Verbose display
set disassembly-flavor intel # Intel syntax instead of AT&T
```


### Basic commands

We can now see various useful commands we can leverage inside gdb:
- `quit`: Exit gdb (lol)
- `break <location>`: Places a breakpoint in the `main()` function
	*When a breakpoint is reached, the debugger pauses the execution. We usually place breakpoints at places where we want to inspect the memory, registries/stack values, etc. In general, we begin by doing `break main`, so we place a breakpoint right before any instruction is executed*
- `run`: Begins the normal execution of the program
- `continue`: Resumes program execution after hitting a breakpoint
	*So, when the execution is stopped by a breakpoint, do not use `run`...*
- `print <variable>`: Display the value stored inside a variable
- `disas <function|address>`: Disassemble machine code into assembly instructions
- `list`: Displays the source code around the current line or at a specific point in the program
	*Note that this only works if extra debugging information were included during compilation (`gcc -g` for instance). In a RE scenario, using `disas` is the way to go...*
- `info registers`: Display all registers and the value they contain.
- `info register <register_name>`: Same as the command above but for a specific register.

Most of those instructions have corresponding shorthand commands.
	*E.g., `info register eip` can be shortened by `i r eip`*

It is also possible to do mathematical operations. The result will be **stored in a temporary variable in the debugger** (called `$1` in the example bellow):
```asm
print $ebp - 4
$1 = (void *) 0xbffff804
x/4xb $1 # Re-using $1. More information on x command bellow (examine part)
0xbffff804: 0xc0 0x83 0x04 0x08
```


### Examine

The `x` command (short for `examine`) is **used to inspect memory at specific addresses**. 
It allows to view raw memory data in various formats, making it useful for debugging low-level issues such as pointer values, array contents, or stack states.
The command expects three arguments: 
- Location in memory to examine
- Format used to display the result (hex by default)
- Size of each memory unit (word by default, so four bytes)
- Number of units we want to examine (1 by default)
``x/<number_of_units><format><unit_size> <address>``

The available display formats (single-letter shorthand) are the following:
- `o` Display in octal
- `x` Display in hexadecimal
- `u` Display in unsigned, standard base-10 decimal
- `t` Display in binary

Example:
```asm
(gdb) x/o 0x8048384
0x8048384 <main+16>: 077042707
(gdb) x/x $eip
0x8048384 <main+16>: 0x00fc45c7
(gdb) x/u $eip
0x8048384 <main+16>: 16532935
(gdb) x/t $eip
0x8048384 <main+16>: 00000000111111000100010111000111
```
Here, `$eip` is equivalent to the value `EIP` contains at that moment.


Available unit sizes:
- `b` A single byte
- `h` A halfword, which is two bytes in size
- `w` A word, which is four bytes in size
- `g` A giant, which is eight bytes in size

Example:
```asm
(gdb) x/8xb $eip
0x8048384 <main+16>: 0xc7 0x45 0xfc 0x00 0x00 0x00 0x00 0x83
(gdb) x/8xh $eip
0x8048384 <main+16>: 0x45c7 0x00fc 0x0000 0x8300 0xfc7d 0x7e09 0xeb02 0xc713
(gdb) x/8xw $eip
0x8048384 <main+16>: 0x00fc45c7 0x83000000 0x7e09fc7d 0xc713eb02
0x8048394 <main+32>: 0x84842404 0x01e80804 0x8dffffff 0x00fffc45
```
> Notice how the units are padded to the right as we enlarge the unit size (0x**c7** -> 0x45**c7***) due to little-endian (least significant byte stored first). gdb is smart enough to reverse the value correctly, that's why we have this behaviour. Demonstration bellow:

```asm
(gdb) x/4xb $eip
0x8048384 <main+16>: 0xc7 0x45 0xfc 0x00
(gdb) x/4ub $eip
0x8048384 <main+16>: 199 69 252 0
(gdb) x/1xw $eip
0x8048384 <main+16>: 0x00fc45c7
(gdb) x/1uw $eip
0x8048384 <main+16>: 16532935
```


However, there is a last format that we did not cover earlier: `i` (instruction). It allows to display the memory as disassembled assembly language instructions:
```asm
(gdb) break main
Breakpoint 1 at 0x8048384: file firstprog.c, line 6.
(gdb) run
Starting program: /home/reader/booksrc/a.out
Breakpoint 1, main () at firstprog.c:6
6 for(i=0; i < 10; i++)
(gdb) i r $eip
eip 0x8048384 0x8048384 <main+16>
(gdb) x/i $eip
0x8048384 <main+16>: mov DWORD PTR [ebp-4],0x0
(gdb) x/3i $eip
0x8048384 <main+16>: mov DWORD PTR [ebp-4],0x0
0x804838b <main+23>: cmp DWORD PTR [ebp-4],0x9
0x804838f <main+27>: jle 0x8048393 <main+31>
(gdb) x/7xb $eip
0x8048384 <main+16>: 0xc7 0x45 0xfc 0x00 0x00 0x00 0x00
(gdb) x/i $eip
0x8048384 <main+16>: mov DWORD PTR [ebp-4],0x0
```

If we encounter values stored as ASCII in our program, we can automatically parse them via `c` or `s`:
```asm
(gdb) x/6ub 0x8048484
0x8048484: 72 101 108 108 111 32
(gdb) x/6cb 0x8048484
0x8048484: 72 'H' 101 'e' 108 'l' 108 'l' 111 'o' 32 ' '
(gdb) x/s 0x8048484
0x8048484: "Hello, world!\n"
```
Here, we see that data string "Hello, world!\n" is stored at memory address `0x8048484`.


Note - Useful command:
```asm
(gdb) x/10i $eip
0x804838b <main+23>: cmp DWORD PTR [ebp-4],0x9
0x804838f <main+27>: jle 0x8048393 <main+31>
0x8048391 <main+29>: jmp 0x80483a6 <main+50>
0x8048393 <main+31>: mov DWORD PTR [esp],0x8048484
0x804839a <main+38>: call 0x80482a0 <printf@plt>
0x804839f <main+43>: lea eax,[ebp-4]
0x80483a2 <main+46>: inc DWORD PTR [eax]
0x80483a4 <main+48>: jmp 0x804838b <main+23>
0x80483a6 <main+50>: leave
0x80483a7 <main+51>: ret
```


****
### Move through instructions

Dedicated commands that works at **Source Code Level**:
- `step`: Executes the current line and stops at the next line of code, stepping into function calls
	*This is very useful if we want to look at a part of the code in details, seeing how each line behaves without having to manually set a breakpoint each time. We simply progress line per line by chaining `step` instructions*
- `next`: Same as `stop` but doesn't step into functions

Corresponding commands that works at **Instruction Level**:
- `nexti`: Executes one assembly instruction. Steps over functions.
- `stepi`: Executes one assembly instruction. Steps into functions.
