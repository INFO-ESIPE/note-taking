[[Java]]
****
**Table of Contents**
```table-of-contents
```

****
## Runtime

Every language defines an execution model. A runtime system is here to implement parts of this model, by providing support during program execution **(runtime support)**.
	*This runtime support is needed by compiled programs, but especially by interpreted ones*

This support is needed for:
1. **Memory management**: `push`/`pop` on the stack, Allocation and Garbage collector on the heap
2. **I/O**: Act as an interface between the program and the FS, network sockets, I/O devices
3. Concurrent/Parallel execution
4. Debugging
5. Code generation and optimisation
	*In Java, this is done by the JIT. We have seen a few examples in the [[04 - Signals#Rendezvous mechanism|concurrent programming class]]
6. ...
> (Wayyyyy) more details provided in "The dragon book" (Compilers: Principles, Techniques, and Tools), which you can download from zlibrary or library genesis :)

The runtime system is made of:
- Compiled code in the executing program
- Code running in other threads/processes during execution
- Language libraries
- OS-provided functionalities
- Interpreter and JVM itself !

The **Java Runtime Environment (JRE)** includes everything needed for our compiled Java programs to run, mostly:
- **The Java Class Library (JCL)**, the basic Java API which contains all basic classes (`java.lang`, `java.io`, `java.util`...)
- **The Java Virtual Machine (JVM)**
> This class will focus on the later. Here are some useful documentation for complementary information:
> https://docs.oracle.com/javase/specs/index.html
> https://blog.jamesdbloom.com/JVMInternals.html


### VM

A Virtual Machine (in the context of a programming language) serves **as an abstraction of the Operating System** we run the program on.
The purpose is to provide the same abstract environment on every machine (WORA).

The VM will be the entity interacting with the executed code (and optimise it on the fly).

Some languages (e.g. C++) doesn't have a VM, only a runtime.
	*C++'s runtime is the standard library, a standard Application Binary Interface (ABI), and the builtin code.*
C++ code is compiled into relocatable objects, then the linker binds the needed libraries to create the executable file (as explained [[Various/Compilers/01 - Fundamentals/01 - Fundamentals#Compiler Structure|here]]) 
![[runtime-cpp.png]]

In Java, source code is compiled into an intermediate code (bytecode) which is platform-independent. Then, the VM loads classes on demand and execute the code.
![[runtime-java.png]]

Each VM has its own architecture, even though they run the exact same language. For instance, the JVM is a **stack machine** (moving values to and from the stack), while Dalvik is a **register machine** (it follows a "Turing complete" register-based architecture, instructions are moved to and from registers).
	*Dalvik is the VM used on Android to run Java code. It was designed to run on machines with low memory. So, it does run Java as well, but will convert `.class` files (containing Java bytecode) into a dedicated `.dex` format.*

Some VMs ensures code validity before execution.
	*JVMs verifies application code, not JDK code (as it's very slow)
	Dalvik verifies code at installation, and then applies hashing on it to ensure it hasn't been modified in the meantime.*


### Interpreter

Two main kinds of interpreters:
1. **Abstract Syntax Tree (AST) Interpreter**: *A.K.A. Tree-Walk interpreters*, language is represented as an abstract tree. It's easy to implement but kind of slow
		*Not much examples really, a few implementations of Lisp mostly*
2. **Bytecode Interpreter**: It's just a loop with a switch, and the later indicates how to handle each bytecode instruction. 
		*JVM, CPython*


### JIT

Several strategies
	- **Just in Time**: Turn into assembly on first call
	- **Tiered**: Often uses two JITs: a quick one and another which is optimised, but slower
		*The second one is only used to optimise code that we often go through*
	- **Mixed-mode**: Interpreter + JIT(s)
		*Oracle Hotspot (An interpreter + 2 JITs)
	- **Ahead of time**: Code is processed through JIT at installation
		*C# Native image generator*


### Garbage Collector

The GC is an algorithm used to **reclaim memory from objects that aren't used anymore**.
	*Unlike C where we have to `free` resources by ourselves, the GC does it for us. The main downside is that we can't exactly predict **when** he will do it.*

It usually triggers:
- When creating a new object (calling `new`) when no memory is available
- Periodically (in minutes for a desktop, in milliseconds for a busy server)

The stereotype that GCs counts references on an object isn't accurate, as this mechanism is nowadays mostly used by old GCs. Modern GCs uses the following logic: We have a graph of objects, and we try to reach them with a **mark phase**. Those we cannot reach are considered dead, and can be freed (**sweep/compact**).

#### Mark Phase

This is the first step. Every object we instantiate is binded to a mark bit (which is, by default, to false).
1. We start at root objects, which are either **references stored in the stack of each threads**, or **static variables**.
2. We now do a traversal of the graph, where each node is an object on the heap
	*A Depth First Search (DFS) approach is often used for this step*
3. We turn the mark bit to true (1) for each object we can reach
> This process can obviously be done while the program is running

Now that our objects are marked, we need to do something with the ones that we could not reach. Two algorithms exists (bellow), but they do share this same mark phase.

#### Mark & Sweep

As the name implies, once objects are marked, we **sweeps unreachable ones** (with a mark bit that remained to false).
	*Equivalent to a `free` on all objects we could not reach.*
Furthermore, we also need to set the mark bit back to false on reachable objects (as we must repeat this procedure several time during runtime, to keep the memory clean).

A (straightforward algorithm) looks like this:
```
foreach (object o in heap) {
	if markedBit(o) {
		// Object could be reached, we simply put the mark bit back to false for
		// future mark phases
		markedBit(p) = false
	} else {
		// Object could not be reached, we free it
		free(o)
	}
}
```
> This method seems good (simple algorithm, no infinite loops), but has a big downside: Reachable objects ends up being separated by a lot of tiny unused memory regions. This **fragmentation** is more and more visible as we repeat this process, which ends up being a huge waste of memory space.
![[fragmentation.png]]

#### Mark & Compact
 
Once objects are marked, we **copy all reachable (marked) objects into another memory zone**.
```
newLocation = 0; // Start of the compacted memory zone

foreach (object o in heap) {
    if markedBit(o) {
        // Object is reachable, move it to the compacted memory zone
        copy(o, newLocation);
        updateReferences(o, newLocation);
        newLocation += sizeOf(o);
        
        // Reset mark bit for future collection cycles
        markedBit(o) = false;
    }
}
```
![[compact.png]]
> This solves the fragmentation issue introduced by the mark & sweep approach, but it requires more memory space to do the copying operation (temporary free zone where we move the objects).


### Generational Garbage Collection

Generational Garbage Collection **divides the heap into generations** based on the hypothesis that **most objects have a short lifespan** and that **objects created at the same time often share similar lifespans**. 
This allows garbage collection to focus more frequently on young objects, improving efficiency:
- **Young Generation** will contains newly allocated objects
	*Collected frequently using **minor collections***
- **Old Generation (Tenured)** will contains long-lived objects promoted from the young generation
	*Collected less frequently using **major collections***

```
  ↓    young    ↓         ↓         old        ↓
```
![[generational.png]]

#### Young Generation

1. **Eden → Survivor** - New objects are initially allocated in the **Eden** space.
    - During a minor collection:
        - Mark reachable objects in the Eden space.
        - Move them to the **Survivor** space.

2. **Survivor → Survivor** - After each minor collection, objects in one Survivor space are copied to the other **Survivor** space if they remain reachable.
    *This swapping continues for several collection cycles, keeping survivors compacted.*
    
3. **Survivor → Tenured** - Objects that survive a certain number of minor collections (defined by a threshold, often called the **tenuring threshold**) are promoted to the **Old Generation (Tenured)**.

#### Tenured Generation

1. **Mark Phase** - During a **major collection**, all objects in the old Generation are scanned to identify reachable ones. Unreachable objects are marked for removal, as we know.

2. **Sweep Phase** - Reclaim the memory occupied by unreachable objects.
 
3. **Optional Compact Phase** - If fragmentation becomes significant in the old Generation (due to the sweep phase), a **mark-and-compact** strategy may be applied to consolidate the memory into a contiguous block.


****
## JVM

JVM is a multi-threaded **stack based** machine. It pops arguments from the operand stack, and push the result on the top of this stack.
	*In details, this operand stack is used to: pass arguments to methods, return a result, store local variables, and store intermediate results while evaluating an expression.*

In fact, the JVM remains an **abstract machine**, as it does not provide much details of implementation (GC algorithm, Internal optimisations...) if we want to design our own VM.
The specification does, however, defines the machine independent `.class` file and its bytecode.
	*So, all JVMs must understand that*

The standard defines the following components:
![[ESIPE/Semestre 3/Java/11 - Runtime/architecture.png]]

As explained [[09 - Reflection#Concept|here]], types are gone after compilation. During runtime, we only manipulate values with dedicated operands which only give clues on what the values are supposed to be (e.g., `iadd`, `fadd`).

The **HotSpot JVM** (among a few other JVMs) profiles the code during interpretation, looking for “hot” areas of byte code that are executed regularly.
These parts are optimised and compiled into native code by the JIT, and is then stored in the **code cache** in non-heap memory.
	*All threads can access it*

The HotSpot JVM includes four [[11 - Runtime#Generational Garbage Collection|generational garbage collection algorithms]], and chose to use the Z Garbage Collector (ZGC) since JDK 11.

