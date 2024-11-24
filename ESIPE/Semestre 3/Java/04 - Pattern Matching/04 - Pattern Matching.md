[[Java]]
****
**Table of Contents**
```table-of-contents
```

****
## Concept

Pattern matching allows—in a single operation—to:
- Test type and value of a data (pattern)
- Extract values from it by using the pattern as a mask
- Execute a specific code on it depending on the type

It mostly comes from functional programming, where we needed to extract values from tuples/product types.
	*OCaml, SML, Erlang*

In Java, it works on classes and records.


****
## Switch

Java's switch was identical to C's (in Java 1.0).
The problem is that C's switch is alien to C itself.
	*In C, if we had several instructions to execute, we put them inside a `{ }` block, but switch is an exception to this as we use the `break` keyword to specify the end of a "block"*
```java
switch(value) {
	case ‘a’:
	case ‘b’:
		// ...
		break; // <- end of instructions
	default:
		//...
}
```
> Cases will be checked in order (so, `case a`, then `case b`, then `default` if none fits). We can only enter one case (so if our value suits several cases, it will only enter the first one it encounters).
> So, the logic is the following: **We go from the most precise to the most generic case**

This is problematic. It introduces two issues:
- **Fallthrough effect** if we forgot a `break`
- The **scope applies on the entire switch**
```java
switch(value) {
	case ‘a’:
		// ...
		break;
	case ‘b’:
		var foo = 3;
		break;
	case ‘c’:
		var foo = 5; // doesn't compile. foo is considered to be already defined
		break;
}
```

### Since Java 14

Switch with arrows are now the way to go.
	*An arrow pointing to a single instruction, or an arrow pointing to a `{ }` block if several instructions must be executed
	Furthermore, a case can accept several values (separated by a comma)*
```java
switch(value) {
	case ‘a’, ‘b’ -> System.out.println("hello !");
	case ‘c’ -> {
		var foo = "hello !";
		System.out.println(foo);
	}
}
```
> The old (C-like) switch still exists—for compatibility purposes—but it is bad practice to use it. An arrow switch cannot be mixed with a C-like switch (`:` case and `->` case).

### Types

A switch accepts the following:
- `byte`, `short`, `char`, `in` (Java 1.0)
	*Switch on the value*
- Enum (Java 5)
	*Switch on `ordinal()`*
```java
enum Color { RED, GREEN, BLUE }

Color color = // …
// This must be exhaustive!
switch(color) {
	case RED -> System.out.println("warning");
	case GREEN, BLUE -> System.out.println("ok");
}
```

- `String` (Java 7)
	*Switch on `hashCode()` (+ `equals()` if collision)*
```java
String kind = // …
Vehicle vehicle; // must be initialised before, so all branches can see it !
switch(kind) {
	case "car" -> vehicle = new Car();
	case "bus" -> vehicle = new Bus();
	default -> throw new AssertionError(); // exhaustive, else it won't compile
}
```

- Types (Java 17)
	*Equivalent to an `if i instanceof` cascade, pretty much*
```java
Object json = // …
switch(json) {
	case JSONObject object - > // ...
	case JSONArray array - > // ...
	default - > // ...
}

// Is equivalent to:
if (json instanceof JSONObject object) {
	// ...
} else {
	if (json instanceof JSONArray array) {
		// ...
	} else {
		// default
	}
}
```

By default, a switch **does not accept null**. If we want to, we must explicitly say it:
```java
case null -> // ...

// For default. This is a workaround as "default" does not accept null values
case null, default -> // ...
```


****
## Patterns and Guards

The following patterns are used:
- **Type Pattern**: `Type variable`; `var variable` (Java 18)
- **Guard Pattern**: `pattern && condition`
- **Parenthesis Pattern**: `( pattern )`
- **Record & Array Patterns** (Java 18)
	- `Type(var x, var y)`, record with two components
	- `Type[] { var x, var y }`, array with 2 values
	- `Type{} { var x, ...}`, array with at least one value

Guards are here to **make a case more granular**
```java
Cake cake = // ...
switch(present) {
	case Cookie cookie && cookie.chunky -> // …
	case Cookie cookie -> // …
	// ...
}
```
> A guarded case must be placed before its non-guarded counterpart. If we placed `case Cookie cookie` before `case Cookie cookie && cookie.chunky`, our code would not compile... And it makes sense actually, just think about it for a second...


****
## Completeness

The compiler requires a switch to be exhaustive. If all possible values aren't covered, then it refuses to compile (or emit a warning).
- `default` covers all values (besides null)
- We call a **total pattern** when all values are covered, including null
- If we cover a finite set of types (Enum, Sealed Interface), the compiler secretly inserts a `default` that crashes the program.

The total pattern looks like this:
```java
Number number = // ...

// Total pattern
switch(number) {
	case Integer i -> // ...
	case Double d -> // ...
	case Number n -> // ... total pattern, switch is exhaustive
}

// Using of "var" instead of the enclosing Type's name, even better !
// Should be added in a future version of Java 
switch(number) {
	case Integer i -> // ...
	case Double d -> // ...
	case var n -> // ... works the same, but we use it to explicitly indicates
				  //     that our switch accepts null values, it's more readable 
}
```


As explained in the first lesson, a sealed interface should not use `default` nor a total pattern. We just cover the current subtypes.
	*This way, if a new subtype is later added by developers, our code doesn't compile and reminds us we have to make modification on the switch*


****
## Switch Expression

The switch can be used as an expression:
```java
Vehicle vehicle;
switch(kind) {
	case "car" -> vehicle = new Car();
	case "bus" -> vehicle = new Bus();
	default -> throw new AssertionError();
}

// Better: use as an expression
var vehicle = switch(kind) {
	case "car" -> new Car();
	case "bus" -> {
		System.out.println("making a bus");
		yield new Bus(); // in case of a block, use the "yield" keyword, as the
						 // "return" keyword is reserved for the function.
	}
	default -> throw new AssertionError();
}; // mind the semicolon
```

