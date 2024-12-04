[[Java]]
****
**Table of Contents**
```table-of-contents
```

****
## Languages and Paradigms

**History of languages:**
1957-1959: First languages
FORTRAN, LISP, ALGOL, COBOL

1967: First OOP language
SIMULA: Class + Inheritance + Garbage Collector

1970-1980: "Paradigmatic" languages
C, Smalltalk, APL, Prolog, ML, Scheme, SQL

1990-2000: Multi-paradigm languages
Python, CLOS, Ruby, Java, JavaScript, PHP, ...


**Paradigms:**
Imperative - Sequence of instruction indicating how to retrieve information by manipulating memory
	*FORTRAN, C, Algol, Pascal*

Declarative - Description of what we have, and what we want (unlike imperative, we do not specify HOW to retrieve what we want)
	*SQL, Prolog*

Functional - Evaluation of expressions/functions where the result does not depend on memory (no side effects\*)
	*Haskell, Caml, Scheme*

Object-Oriented (OOP) - Reusable, abstract units that control side effects
	*C++, Objective-C, Modula*

Data-Oriented - Data is more important than the code
	*Haskell, SML, Caml, F#*

\* Side effect occurs when a function or expression modifies some state outside its scope, such as changing a variable, updating memory, or interacting with external systems.
In Functional paradigms, we focus on **immutability** to ensure predictability. 


A paradigm does not exclude another (as we can see above, Haskell is both data-oriented and functional).
In fact, most modern languages are **multi-paradigms** as they allow to mix those programming styles.


****
## Java ?

Java is multi-paradigm:
- **Imperative**: Sequential code, loop, *mutable* collection
- **Functional**: Lambda, Record, *immutable* collection
- **Declarative**: Stream, RegEx
- **OOP**: Class (encapsulation), Subtyping, Polymorphism/Late Binding
- **Data**: Record, Sealed Interface, Pattern Matching


Java is actually a set of distributions
	*Oracle Java, RedHat OpenJDK, Amazon Coretto...*
A distribution must pass the Test Compatibility Kit (TCK) to be named "Java".

Ultimately, Java is two things:
- **The language**: Java Language Specification (JLS)
- **The Platform**: Java Virtual Machine Specification (JVMS)
> This distinction is important, as the Java language is not the only language to run on the Java platform
	*Scala, Kotlin, Clojure and Groovy relies on it as well*

Furthermore, the JVM and the language does not share the same features.
	*The VM does not know Varargs, Generics, Checked Exceptions, Records, Internal classes, Enumerations...*


As explained [[Various/Compilers/01 - Fundamentals/01 - Fundamentals#Java|here]], Java code is compiled into an intermediate language called **bytecode** before execution. This bytecode is then interpreted during **runtime on the JVM**.
	*The JIT compiler is also here at execution to make performance improvements. More details on Java runtime [[11 - Runtime|here]]*
This allows Java to follow the Write Once Run Anywhere (WORA) paradigm
	*Any machine can consistently run a java program as long as the Java Platform is installed on it.*


Java evolves — like all maintained programming languages actually lol — as software needs evolves. An important thing to understand here is that **Java follows a backward-compatibility philosophy**, which (mostly) means two things:
- **Bytecode compatibility**: Bytecode compiled with an older Java version can run on a modern JVM
- **Source code compatibility**: A source code that was written on an old Java version can still be compiled in recent versions.
	*This means that the API must be kept. If a class (or one of its public methods) become deprecated, it must be kept !!!!*

> For this class, we will use the OpenJDK 22, which includes a compiler `javac`, the JVM, and other useful materials. 


****
## OOP

Java provides the following OOP concepts:
- **Encapsulation (classes)**: We separate the public API from its private internal implementation

- **Subtyping (interfaces)**!: Various classes can be considered as the type they implement
```java
// Cat is a subtype of Animal. "myCat" is considered as both a Cat and an Animal
Animal myCat = new Cat();
```

- **Late Binding**: (During execution) calls the method of the correct implementation
	*`myCat.sound()` will call `Cat::sound`, and not `Animal::sound`*


### Encapsulation

We say that a class encapsulates (protect) its data.
The public API is what external classes can access in order to interact with the class
	*public/protected methods*
The implementation is the hidden part that defines how the API works under the hood 
	*private fields, private methods, private inner classes*

We can make a field **immutable** by specifying the `final` keyword.
	*A definitive value must be assigned to the field at object's creation. It means we can not reassign this variable.*

==Fields of a class SHOULD ALWAYS BE PRIVATE== (and final if possible).
Only exception for constants (`static final`), which can be public if needed.

**We must follow this principle:**
- Everything a class controls (its own fields and methods) is trustworthy
- Everything coming from the outside is potentially risky
	***Design by contract**: An object's state must always be correct. We will check that via pre-conditions (and throw an `Exception`, if it doesn't satisfy our needs)*


### In practice

Let's consider everything coming from the outside will attempt to jeopardise our class.
Every public access point must be protected by **pre-conditions** and **defensive copies**, as we can not trust what others passes to us.

Let's say I want to organise french classes for Erasmus students, but i don't want french students to be able to join it
	*they already speak the language, so whats the point?*

```java
// We don't control this record, we only use it
record Student(String name, String nationality) {
	public Student {
		Objects.requireNonNull(name);
		Objects.requireNonNull(nationality);
	}

	public boolean isFrench() {
		return nationality.equalsIgnoreCase("french");
	}
}

// This is our class
class FrenchClass {
	private String teacher;
	private List<Student> students = new ArrayList<>();;
	
	public FrenchClass(String teacher) {
		this.teacher = teacher; // oops 1
	}

	public void register(Student newStudent) {
		students.add(newStudent); // oops 2, 3
	}

	public List<Student> students() {
		return students; // oops 4
	}
}
```

A few problems here. First, **our public API has no protection**.
We need to add pre-conditions that ensures what we manipulate is reliable:
```java
class FrenchClass {
	private String teacher;
	private List<String> students = new ArrayList<>();;
	
	public FrenchClass(String teacher) {
		// Any object can be null. Always ensure it isn't when one
		// is passed as a parameter
		Objects.requireNonNull(teacher);
		this.teacher = teacher;
	}

	public void register(Student newStudent) {
		Objects.requireNonNull(newStudent);
		if (newStudent.isFrench()) {
			throw new IllegalArgumentException("Only Erasmus students allowed");
		}
		
		students.add(newStudent);
	}

	public List<Student> students() {
		return students;
	}
}
```

This is better, but there is a subtle problem that allows an user to break the encapsulation. Here, it is possible to bypass the nationality restriction:
```java
var class = new FrenchClass("M. Maurice");
var students = class.students();
students.add(new Student("goobert", "french")); // oops
```
> In fact, the `students()` method returned the **direct reference** to our internal collection. This allows anyone to manipulate it from the outside.
	*This way, we could add a french student to the class even though our `register()` method doesn't allow it. We bypassed the encapsulation...*

We need to make a **defensive copy** each time we **return an internal collection (or any mutable class)**, or each time **we accept one from outside**:
```java
class FrenchClass {
	private String teacher;
	private List<String> students = new ArrayList<>();
	
	public FrenchClass(String teacher) {
		Objects.requireNonNull(teacher);
		this.teacher = teacher;
	}

	public void register(Student newStudent) {
		Objects.requireNonNull(newStudent);
		if (newStudent.isFrench()) {
			throw new IllegalArgumentException("Only Erasmus students allowed");
		}
		
		students.add(newStudent);
	}

	public List<Student> students() {
		return List.copyOf(students); // defensive copy
	}
}
```


The only big issue that remains here is about our implementation. The class' properties can (and should) be final here.
	*it is always a good practice to define all of our fields as final, and remove it later on if we realise we can not keep it immutable. In this case, the teacher name will not be changed after it was defined, and the collection remains the same (we modify it obviously, but don't REASSIGN a new collection)*

Furthermore, we could use an `HashSet` instead of a List (which avoids duplicate students issue).

Let's fix all of this:
```java
class FrenchClass {
	private final String teacher;
	private final Set<Student> students = new HashSet<>();
	
	public FrenchClass(String teacher) {
		Objects.requireNonNull(teacher);
		this.teacher = teacher;
	}

	public boolean register(Student newStudent) { // signature changed for set
		Objects.requireNonNull(newStudent);
		if (newStudent.isFrench()) {
			throw new IllegalArgumentException("Only Erasmus students allowed");
		}
		
		return students.add(newStudent); // false if already registered
	}

	/*
	Note: External classes are not aware that we use a HashSet to store our 
	students. It is good convention to return it as a(n immutable) List
	*/
	public List<Student> students() {
		return List.copyOf(students);
	}
}
```

Now our class is well-encapsulated and has a decent implementation (we could make pre-conditions for students/teacher names as well if we were meticulous).
	*Even though both our properties are final, we don't make a Record out of it. Even if we could, the Record's philosophy is that we NEVER should be able to modify it. 
	Since our collection is mutable, we respect this convention and don't use a record.*


****
## Types

Java is **statically-typed**, so it expects its variables to be declared before they can be assigned values.
	*Both primitives and classes are considered as types*

**Primitives:**
- boolean (true/false)
- byte (signed)
- char (unsigned 2 bytes)
- short (signed 2 bytes)
- int (signed 4 bytes)
- long (signed 8 bytes)
- float (4 bytes)
- double (8 bytes)
**Object types:**
- Reference (opaque representation)

We can declare the type:
- **Implicitly** (better)
```java
var y = 10; // int
var point = new Point(5, y); // Point
```

- **Explicitly**
```java
int x = 5;
/*
↓Type↓           ↓Class↓ */
Point point = new Point(x, 10);
```

The **type** (on the left) is **for the compiler**.
	*It is not necessarily kept during runtime.*
The class (on the right) is for the **runtime environment**
	*Allows to know the size on the heap (for a `new`, we need to know how much space we must allocate in memory to store the object)*


### Boxing

Primitive types have object counterparts called **Wrapper Types**
	`int` -> `Integer`
	`long` -> `Long`
	...
**ALWAYS USE PRIMITIVES WHEN ITS POSSIBLE**, as it is way more performant
	*Main exception to this rule will be for collections, as they require usage of Wrappers*

> This separation is actually the biggest design flow of Java. It should be the JVM's job to decide if an `Integer` or an `int` should be used depending on the context, not the developer's. This forces Java to do boxing operations, which destroys performance.
> 	 Kotlin fixed this issue

Boxing:
```java
// Auto-box
int i = 3;
Integer large = i;

// Auto-unbox
// Integer large = new Integer(3); NO !
Integer large = 3;
int i = large;
```
> Mind the commented line: Actually, a Wrapper's identity (memory address) is not well defined...
> We don't use `new` on a wrapper. It should even result in a compilation error in future Java versions

Another issue with java is about comparing values between Wrappers. The current implementation only shares values on a single signed byte (from -128 to 127).
So, we get the following stupid situation:
```java
Integer val = 3;
Integer val2 = 3;
val == val2 // true

Integer val = -200;
Integer val2 = -200;
val == val2 // can be true or false, we don't know (lol ???)
```

So, we never use `==` and `!=` to compare two Wrappers. Use `equal()`, like for all objects.
	*Actually you should get a warning if you attempt to do so*


****
## Subtyping

The Liskov principle explains that if we manipulate two objects from two different classes at the same location, those classes are subtypes.

Example:
```java
void foo() {
	if (/*...*/) {
		v = new Car();
	} else {
		v = new Bus();
	}
	// ...
	v.drive(); // type Car | Bus
}
```

We can not define a `A | B` type, so we do this via an **interface** and specify that both A and B implements this interface.
```java
interface Vehicle { void drive(); }

record Car() implements Vehicle { public void drive() { /* ... */ } }
record Bus() implements Vehicle { public void drive() { /* ... */ } }
```
> Interfaces are here for **typing**, not execution.

But how does the JVM knows how to call the correct method ? Example:
```java
void driveAll(List<Vehicle> vehicles) {
	for(Vehicle vehicle: vehicles) {
		vehicle.drive(); // Compilation: Vehicle::drive()
						 // Runtime: calls Car::drive() then Bus::drive()
	}
}

List<Vehicle> list = List.of(new Car(), new Bus());
driveAll(list);
```

Late binding allows this.


****
## Late Binding

A Java Object is simply a memory pointer (reference) to an address we cannot directly access to.
	*Only the Garbage Collector can move Objects directly in memory*

An object's content is **allocated on the heap** (via the `new` instruction). It is **freed by the Garbage Collector (GC) in a non-predictable manner**.
	*So, the stack just contains an address pointing to a zone in the heap (where the object's body is).*
It looks like this:
![[ESIPE/Semestre 3/Java/01 - Fundamentals/memory.png]]

An object obviously possesses its own fields, but also a **header**.
This header includes a pointer towards the object's class, hashCode, state when it is Garbage Collected, etc...
```java
class Car {
	// pointer towards the class +
	// hashCode + lock + GC bits = header of around 64bits (depending on
	// the VM and architecture)
	int seats; // int => 32bits => 4 bytes
	double weight; // double => 64bits => 8 bytes
}
```

**All objects are aware of their corresponding class!**


So far so good, but what about the methods ?
Well, an instance method (executed on the object) is represented inside the object's heap as a **function pointer**.
This points to the **class' Virtual Table (vtable)**. All classes have a corresponding vtable, which is a mechanism used by many OOP languages—including Java—to support **dynamic method dispatch**.
	*All non-static and non-final methods of a class are stored in a vtable. Static and final methods are resolved at compile-time (static binding)*
![[vtable.png]]

If a method has an `Override`, their index in the vtable is the same
	*So, the method `.toString()` of the two classes shares a same index (0 here)*
![[vtable-override.png]]
> In fact, the `vehicle.toString()` call will be transformed into `vehicle.vtable[index]()` by the JVM (so, at runtime)


### Static fields

A constant (`#define` in C) is a **static final field**.
```java
record Color(int red, int green, int blue) {
	Color {
		if (red < MIN || red > MAX) { throw new IAE(...); }
		// ...
	}

	static final int MIN = 0;   // constant "MIN"
	static final int MAX = 255; // constant "MAX"
}
```

A static field (owned by the class itself, not an instance of it) is stored in the class' vtable:
![[static.png]]


****
## Inheritance

Inheritance implies **subtypes** (like an interface), **copy of fields and methods** (pointers) and **method override**.

This introduces a serious problem: Copying members implies a **strong relationship** between the parent and child(ren) class(es).
- What if we update the parent class but don't update the children ?
- What if the parent class is non-mutable but the children are ?

This is the main issue with inheritance, it makes the code **really hard to maintain**.
	*Also, as explained in [[02 - SOLID#**L**iskov substitution principle|design pattern]] classes, inheritance breaks the SOLID principle, which is an important concept in OOP.*

It is better to **delegate** than use inheritance:
```java
// This code is wrong! addAll() allows to store null students
public class StudentList extends ArrayList<Student> {
	public void add(Student student) {
		Objects.requireNonNull(student);
		super.add(student);
	}
}

// This code is correct. It uses delegation.
public final class StudentList {
	private final ArrayList<Student> students = new ArrayList<>();
	
	public void add(Student student) {
		Objects.requireNonNull(student);
		students.add(student);
	}
}
```


==INHERITANCE IS OVERALL A BAD DESIGN CHOICE, IT IS BETTER TO USE INTERFACES INSTEAD. THERE ARE VERY FEW SCENARIOS WHERE INHERITANCE IS A GOOD SOLUTION TO SOLVE YOUR PROBLEMS==
> In fact, most modern languages (Go, Rust) doesn't support inheritance for this reason.


### Abstract classes

Are we allowed to use abstract classes ?
- Never instead of an interface! An abstract class **mustn't be used as a type**
- If we need a way to share code, then yes, but:
	- The abstract class should only be used as an inner (so, **never public**)
	- The abstract class and its sub-classes **must be in the same package**


****
## Pattern Matching

it is possible to write a same code by either using **late binding** or **an ugly cascade of `instanceof`**:
```java
// Late Binding
interface Vehicle {
	void drive();
}
record Car() implements Vehicle {
	public void drive() { /* 1 */ }
}
record Bus() implements Vehicle {
	public void drive() { /* 2 */ }
}

// Big heap of shit
interface Vehicle { }
record Car() implements Vehicle { }
record Bus() implements Vehicle { }
// ...
public void drive(Vehicle vehicle) {
	if (vehicle instanceof Car car) {
		/* 1 */
	} else if (vehicle instanceof Bus bus) {
		/* 2 */
	}
	throw new ISE("Scenario not planned because my code stinks"); // bloody hell
}
```

A mechanism called **pattern matching** will check **exhaustiveness of a switch** when it is used **on a sealed interface**.
	*A sealed interface only allows a closed set of classes to implement it, unlike an opened interface where any class can. Since this subset is closed, we can check exhaustiveness...*

It looks like this:
```java
sealed interface Vehicle permits Car, Bus { } // seal
record Car() implements Vehicle { }
record Bus() implements Vehicle { }
// ...
public void drive(Vehicle vehicle) {
	switch(vehicle) {
		case Car car -> /* 1 */
		case Bus bus -> /* 2 */
		// NO DEFAULT !
	}
}
```

==**Why no default ?**==
Pattern matching fits Data-Oriented programming, where data is more important than the code.
If our huge project with a lot of files evolves, and allows the interface to permit new classes, we need our switch cases to deal with those new types too.

If there is no default, our program will refuse to compile until we fix the lack of exhaustiveness.
	*This is a very good behaviour, it serves as a reminder of every point where we need to change the code. 
	The usual problem is the following: devs set a default that crashes the program. When they add new types in the `permit` clause of the interface, they forgot to deal with those new types in those `switch`. In general, the new type will fall into the `default` scenario, which is often an `Exception` we throw to alert that we encountered a type we did not expect.
	**IT IS BETTER FOR OUR PROGRAM TO NOT COMPILE AS A REMINDER OF PLACES WHERE WE HAVE TO MAKE CORRECTIONS, INSTEAD OF FORGETTING ABOUT IT AND HAVING RUNTIME (PRODUCTION) ERRORS.
	A COMPILE-TIME ERROR IS WAY BETTER THAN AN UNPREDICTABLE RUNTIME ERROR** 

