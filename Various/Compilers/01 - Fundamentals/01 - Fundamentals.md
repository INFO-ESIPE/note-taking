****
**Table of Contents**
```table-of-contents
```

****
## Compiler ?

All softwares are written in some programming languages, which are notations for describing computations to people and to machines.
But, before a program can be run, it first must be translated into a form in which it can be executed by a computer. That's the task of the compiler.

A compiler is a program that can read a program in one language—the source
language—and **translate it into an equivalent program in another language**—the target
language.
	*If this target program is executable machine-language program, it then can be called by the user to process input and produce output.
	If the compiler's purpose is to translates a high-level language into another high-level language (instead of translating it to executable code), it is called a source-to-source translator.*

To put it simply, it's a **language processor**


****
## Interpreter

An interpreter is another kind of language processor. Instead of producing a target
program as a translation, an interpreter **appears to directly execute the operations specified in the source program** on inputs supplied by the user.

A compiled code is usually much faster than when processed by an interpreter. 
An interpreter, however, can usually give better error diagnostics than a compiler, because it executes the source program **statement by statement**. 


****
## Java

Java combines both compilation and interpretation:
1. Source program (written in java) is first compiled into an **intermediate form** called **bytecode**
2. This bytecode is then interpreted by the JVM (Write Once Run Anywhere)
3. Sometimes, another compiler called the **JIT (Just In Time compiler)** will improve the performance of the program by **translating frequently executed bytecode into native machine code at runtime**. 
	*Thus eliminating the need for repeated interpretation of the same bytecode*

![[java.png]]


****
## Compiler Structure

Sometimes, the compiler itself is not enough to create our executable target program.
	*For instance, a source program may be divided into modules stored in separate files. The task of collecting the source program is sometimes entrusted to a separate program called a **preprocessor***


### Basic language-processing system 

A common processing system is the following: 
1. The compiler will produce an assembly language program as its output (instead of machine code).
	*Asm language is both easier to produce and debug*
2. This asm is then processed by an **assembler** which will output the machine code
3. If the program is large, divided in modules/pieces, or rely on libraries, we will also need the help of a **linker**, which will link our relocatable machine code with other relocatable objects
	*If the linker sees that our code relies on code present in another file (an external memory address), it will resolve it and link it to our program so it works properly*
4. Finally, the **loader** will put all of our executable objects into memory for execution

![[processing.png]]


### Breaking down the mapping

Actually, to output a target program that is semantically equivalent to the source one, the compiler does two actions: **analysis** and **synthesis**. 

#### Analysis (front-end)
The analysis part is responsible of **detecting structural anomalies** in the source code.
	*This mostly works via regular expressions*
If this part detects issues with the code's structure, it informs the user and stops the compilation.
	*It is important for the analysis part to provide a very detailed and comprehensive error message to the developer, so he quickly understand the issue and fix his code*

To do all of this, the program is broken down into fundamental components, which results in a simplified, machine-independent version of the program called the **Intermediate Representation (IR)**. 
Various information (variables, constants, functions) about it are then stored in a structured data format called the **symbol table**.

#### Synthesis (back-end)
This is where the target program is built, based on both the IR and information contained in the symbol table.


### Phases

If we look closer, we can see a compiler operates under a sequence of **phases**, where each **transforms one representation of the source program into another**.
	*All phases exploits the symbol table*

![[phases.png]]


****
## Breaking down the phases

Let's have a deeper look into the aforementioned phases, one by one...
We will focus on the following simple instruction:
```c
position = initial + rate * 60;
```

### Lexical Analysis 

This is the first phase (A.K.A. scanning). The analyser reads the code (stream of characters) and **groups the characters into meaningful sequences** called `lexemes`.

This is then passed to the next phase (syntax analysis).

Lexems looks like this:
	`<token-name, attribute-value>`
Where `token-name` is an abstract symbol that is used during syntax analysis, and `attribute-value` points to an entry in the symbol table.

Let's break down our instruction:
1. `position` is a lexeme that would be mapped into a token `<id , 1>`, where id is an abstract symbol standing for identifier and 1 points to the symbol-table entry for `position`
	*The symbol-table entry for an identifier holds information about the identifier, such as its name and type.* 
	
2. The assignment symbol `=` is a lexeme that is mapped into the token `<=>`. Since this token needs no attribute-value, we have omitted the second component. 
	*We could have used any abstract symbol such as assign for the token-name, but for notational convenience we have chosen to use the lexeme itself as the name of the abstract symbol.*

3. `initial` is mapped into `<id , 2>`, where 2 points to the symbol table entry for `initial`

4. `+` is mapped into `<+>`

5. `rate` is mapped into `<id , 3>`, where 3 points to the symbol table entry for `rate`

6. `*` is mapped into `<*>`

7. `60` is a lexeme that is mapped into the token `<60>`

We would obtain the following (blanks in between lexemes only here for readability):
`<id, 1> <=> <id, 2> <+> <id, 3> <*> <60>`


### Syntax Analyser

Syntax analysis (or **parsing**) is the second phase. Based on the lexical analysis output, this step will **create a syntax tree** representation that depicts the grammatical structure of the token stream.

This tree shows the order in which the operations in the assignment are to be performed, represented by **nodes**.
For instance, the node `*` will have a left **child** `<id, 3>`, and an integer right child `60` 
	*All of this follows usual conventions of arithmetic which tell us that some operations have higher precedence than others (multiplication over addition for instance)*

We will obtain the following tree:
```
		=
      /   \
<id, 1>	   +
		 /   \
   <id, 2>    *
			/   \
	  <id, 3>  	 60
```


### Semantic Analysis

This part will use the syntax tree built by the previous step (along with the symbol table's information, as always) to **ensure semantic consistency between the programming language definition and the source program**.
	*It checks that we respect the basic syntax of the programming language we used in our code*

The major role of this phase is **type checking**. The compiler will ensure that each operator has matching operands.
	*E.g., using a float or a string value as an array index (it should be an integer)*
This step is also in charge of gathering those types and saving them for later use.
	*It is saved either in the tree itself, or in the symbol table*

Furthermore, some type conversions called **coercions** can be carried.
	*If there is a multiplication between a float and 60 (integer), the later will be turned into a floating-point number*


So ultimately, this step has ensured that our syntax and types where correct, saved those types somewhere, and applied coercion when needed on the tree. We get the following:
```
		=
      /   \
<id, 1>	   +
		 /   \
   <id, 2>    *
			/   \
	  <id, 3> inttofloat
	             | 
				60.0
```


### Intermediate Code Generation

After the two previous phases are completed, most compilers generate an explicit low-level or **machine-like intermediate representation**, which we can think of as a program for an
abstract machine.

This intermediate representation should have two important properties: 
- Easy to produce
- Easy to translate into the target machine

Most compilers (like Clang or LLVM) will use **Three-Address Code (TAC/3AC)** to represent instructions for this phase.
	*With 3AC, each instruction is broken down into up to three operands (typically assignments and operations. The instructions will look assembly-like (operands acting LIKE registers)*
> The overall point is to simplify complex calculations into step-by-step instructions

We will obtain a result that looks like this:
```
t1 = inttofloat(60)
t2 = id3 * t1
t3 = id2 * t2
id1 = t3
```

There are a few things to notice in this:
1. **One Operator per Instruction**: Each three-address instruction performs only **one operation** at a time
2. **Temporary Variables**: The compiler introduces **temporary names** (like `t1`, `t2`, and `t3`) to store intermediate results
3. **Fewer than Three Operands**: While called 3AC, some may have fewer operands depending on the operation
	*For instance, `t1 = inttofloat(60)` only has two "addresses" (`t1` and `60`), since a number conversion does not require a third operand*


### Machine-Independent Code Optimisation

This phase attempts to improve the intermediate code it increases the odd of ending with a better target code. 
	*Usually, by "better" we mean faster, but we can also aim at shorter code, or code consuming less power (a huge loop incrementing a value instead of doing a direct assignation of the final value).*

The optimiser deduces that the conversion of 60 from integer to floating point can be done once and for all at compile time. The `inttofloat` operation can be eliminated by replacing the integer 60 with it's floating-point counterpart (60.0).

This, coupled with other optimisations, result in the following:
```
t1 = id3 * 60.0
id1 = id2 + t1
```


### Code Generation

This part takes as input the intermediate language we obtained from the previous step, and maps it into the target language.

If the target language is machine code, registers or memory locations are selected for each of the variables used by the program. Then, the intermediate instructions are translated into sequences of machine instructions that perform the same task. 

A crucial aspect of code generation is the judicious assignment of registers to hold variables. For example, using registers `R1` and `R2` , the intermediate code might get translated into:
```
LDF R2, id3 // loads the contents of address id3 into register "R2" 
MULF R2, R2, #60.0 // multiplies it with floating-point constant 60
LDF R1, R1, R2
STF id1, R1
```

The first operand of each instruction specifies a destination. 
	*The F in each instruction indicates we manipulate floats*


Here is a summary of our journey:
![[summary.png]]