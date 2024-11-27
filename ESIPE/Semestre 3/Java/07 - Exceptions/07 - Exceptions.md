[[Java]]
****
**Table of Contents**
```table-of-contents
```

****
## Concept

Exceptions can be quite tricky to use correctly.
We will try to solve those basic questions in a correct manner:
- What is the purpose of Exceptions ?
- How to handle them ?
- How to correctly propagate an Exception ?
- What is "Design by contract" ?
	*We briefly explained this one already, but we will add a few more details*

An `Exception` is a mechanism that **reports a situation to the calling method (recursively)**. 
	*So, if one error throws an exception, it will propagate to the method that called it, and it will keep propagating until we reach the main function*

==An Exception is not necessarily an error (e.g., `InterruptedException` for threads==

```java
static class Option {
	Path outputPath;
	boolean all;
	int level;
}

private static Option parseArguments(String[] args) {
	Option options = new Option();
	Map<String, Consumer<Iterator<String>>> actions = Map.of(
		"-output", it -> options.outputPath = Path.of(it.next()),
		"-all", it -> options.all = true,
		"-level", it -> options.level = Integer.parseInt(it.next()));

	for (Iterator<String> it = List.of(args).iterator(); it.hasNext();) {
		actions.get(it.next()).accept(it);
	}
	if (options.outputPath == null) { /* FIXME */ }
	return options;
}

public static void main(String[] args) {
	Option options = parseArguments(args);
	// ...
}
```
> This code can crash at four different locations (+ an awful `FIXME`)

An Exception possesses:
- A type (e.g., `NoSuchElementException`, `NullPointerException`...)
- An error message
- A **stacktrace**, which is the **sequence of methods leading to the Exception**

Let's execute our code above, and analyse the stacktrace:
```bash
java Example -output

Exception in thread "main" java.util.NoSuchElementException   # Exception
at java.util.AbstractList$Itr.next(.../AbstractList.java:377) # Specific location
at exception.Example.lambda$0(Example.java:20)                # Call sequence ...
at exception.Example.parseArguments(Example.java:24)          # ...
at exception.Example.main(Example.java:32)                    # ...
```

We got an error with our Iterator. `AbstractList.Itr.next()` is a JDK method, we must read the documentation, which is:
	`E next()`: Returns the next element in the iteration`
	Returns: the next element in the iteration
	**Throws: `NoSuchElementException` - if the iteration has no more elements**

We can add an intermediate method to give a bit more context **to the developers**:
```java
private static Path parseOutputPath(Iterator<String> it) {
	if (!it.hasNext()) {
		throw new IllegalArgumentException("output path requires an argument");
	}
	return Paths.get(it.next());
}

private static Option parseArguments(String[] args) {
	// ...
		"-output", it -> options.outputPath = parseOutputPath(it); // Change here
	// ...
}
```

Our stacktrace is now more precise on the real issue:
```
Exception in thread "main" java.lang.IllegalArgumentException:
output path requires an argument
at exception.Example.parseOutputPath(Example.java:19)
at exception.Example.lambda$0(Example.java:27)
at exception.Example.parseArguments(Example.java:31)
at exception.Example.main(Example.java:39)
```

However, this remains a bug that crashes the application. The end user should not encounter this...


### Catch

We can capture the Exception with a `try/catch` block. Let's do it in our main function:
```java
public static void main(String[] args) {
	Option options;
	try {
		options = parseArguments(args);
	} catch(IllegalArgumentException e) {
		System.err.println(e.getMessage());
		return;
	}
	// ...
}
```
> `catch()` is doing an `instanceof` on the Exception's type, so it only captures the type we are interested in.

We can make a multi-catch (a `catch` clause can intercept several kinds of Exception), and we can also chain the catch clauses. They will be checked in order:
```java
try {
	options = parseArguments(args);
// Checked First 
} catch(IllegalArgumentException e) {
	System.err.println(e.getMessage());
	return;
// Checked Second, is doing multi-catch also with the "|"
} catch(NoSuchElementException | NumberFormatException e) {
	System.err.println("invalid argument parsing");
	return;
}
```
> Just like for switch cases, we must be careful to organise errors by hierarchy if one is a parent of another (e.g., `NoSuchElementException` inherits from `RuntimeException`, so if you want to catch both, catch `NSEE` first...)


### Creating a custom Exception

Sometimes our code introduces some complexity that needs its own Exceptions to be dealt with correctly:
```java
// More details on the extends RuntimeException later
public class OptionParsingException extends RuntimeException {
	public OptionParsingException(String message) {
		super(message);
	}
	public OptionParsingException(Throwable cause) {
		super(cause);
	}
	public OptionParsingException(String message, Throwable cause) {
		super(message, cause);
	}
}
```
> We need to implement the three constructors!

The constructor taking the cause as a parameter allows for an easier debugging as it indicates the Exception which caused the problem:
```java
private static int parseLevel(Iterator<String> it) {
	if (!it.hasNext()) { 
		// String constructor (1)
		throw new OptionParsingException("level requires an argument"); 
	}
	
	try {
		return Integer.parseInt(it.next());
	} catch (NumberFormatException cause) {
		// String + Throwable constructor (3)
		// Exceptions are chained
		throw new OptionParsingException(
			"level argument is not an integer", cause);
	}
}
```

> This allows to **chain the Exceptions**, so both will result in the stacktrace:
```
Exception in thread "main" exception.OptionParsingException: level
argument is not an integer
at exception.Example.parseLevel(Example.java:31)
at exception.Example.lambda$2(Example.java:40)
at exception.Example.parseArguments(Example.java:42)
at exception.Example.main(Example.java:51)
Caused by: java.lang.NumberFormatException: For input string: "foo"
at java.lang.NumberFormatException.forInputString(...:65)
at java.lang.Integer.parseInt(java.base@9-ea/Integer.java:695)
at java.lang.Integer.parseInt(java.base@9-ea/Integer.java:813)
at exception.Example.parseLevel(Example.java:29)
```

In any case, the `.initCause()` method allows to bind a cause to our Exception even if it does not have a constructor that accepts a cause.
```java
private static int parseLevel(Iterator<String> it) {
	// ...
	try {
		return Integer.parseInt(it.next());
	} catch (NumberFormatException cause) {
		var e = new OptionParsingException("level argument is not an integer");
		e.initCause(cause); // bind
		throw e;
	}
}
```
> We can only call `initCause()` once per instance.


****
## Exceptions and the Compiler

As briefly explained [[07 - Interruptions and Exceptions#Reminder|here]] For the compiler, only two kinds of Exceptions exists:
- **Checked**: Exceptions we must check. They require the use of `throws`.
- **Unchecked:** Exceptions that **does not need to/shouldn't be captured**, mostly happens due to programming errors or Errors. 

So, any Exception which inherits from `Throwable` must be checked, except the ones inheriting from `RuntimeException` and `Error`.

Obviously, a class must inherits from `Throwable` for it to be used along with the `throws` directive.
Hierarchy of throwables:
![[throw.png]]


### Throwable usage

An Exception is due to external conditions (`IOException`, `InterruptedException`). We must catch those and deal with them in order to have a stable program.

An Error (`OutOfMemoryError`, `InternalError`) **MUST NEVER BE CATCHED**.

A `RuntimeException` describes programming errors, mostly...


This is a lot of stuff to catch, our odds of having a decent code are collapsing...
Usually, a checked exception is declared with the `throws` directive, so it can be propagated:
```java
// We declare in the signature that our method can propagate an IOException
// that should be handled by the calling method
private static long parseTime(Iterator<String> it) throws IOException {
	if (!it.hasNext()) {
		Path current = Paths.get(".");
		
		// IOException can be raised here
		return Files.getLastModifiedTime(current).toMillis();
	}
	// ...
}
```


### Tunneling

The problem (encountered [[08 - Serveur TCP Non-bloquant|here]]) is that methods throwing Checked Exceptions does not behave well with the rest of the code:
```java
private static long parseTime(Iterator<String> it) throws IOException {
	if (!it.hasNext()) {
		Path current = Paths.get(".");
		return Files.getLastModifiedTime(current).toMillis();
	}
	// ...
}

private static Option parseArguments(String[] args) {
	Option options = new Option();
	
	Map<String, Consumer<Iterator<String>>> actions = Map.of(
		"-time", it -> options.time = parseTime(it), 
		/* ... */);

	for(Iterator<String> it = List.of(args).iterator(); it.hasNext();) {
	 	Consumer<Iterator<String>> consumer = actions.get(it.next());
	  	consumer.accept(it);
	}
	
	return options;
}
```
> This code does not compile. The `it -> options.time = parseTime(it)` lambda is converted into a `Consumer<...>`, however, the `accept()` method of the `Consumer` does not indicate that it propagates `IOException` (nor any other checked exception)

We can solve this issue by putting the Checked Exception inside an Unchecked Exception, and retrieves the real cause of the Exception in the catch clause:
```java
private static long parseTime(Iterator<String> it) /* No throws anymore here */ {
	try {
		// ...
	} catch(IOException e) {               // Catches the Checked Exception
		throw new UncheckedIOException(e); // Throws it as an Unchecked Exception
	}
}

private static Option parseArguments(String[] args) throws IOException {
	Option options = new Option();
	Map<String, Consumer<Iterator<String>>> actions = Map.of(
		"-time", it -> options.time = parseTime(it), 
		/* ... */);

	for(Iterator<String> it = List.of(args).iterator(); it.hasNext();) {
		Consumer<Iterator<String>> consumer = actions.get(it.next());
		try {
			consumer.accept(it);
		} catch(UncheckedIOException e) { // Tunneling
			throw e.getCause();
		}
	}
	// ...
	return options;
}
```


### Summary

If the error cannot be handled at this point, we use a throws declaration.
If the error can be handled, we use a catch block.
In the `run()` method of a Runnable or in the main method, we must use a catch block.

We aim to handle the error as low as possible in the execution stack to avoid duplicating catch block code.


****
## Exceptions and I/O

The `try-finally` block allows to force the execute of some code, even if an Exception is thrown in the `try`:
```java
BufferedReader reader = Files.newBufferedReader(path);
try {
	doSomething(reader);
} finally {
	// Even if a problem happens with "doSomething", we close the resources
	reader.close();
}
```
> This is very useful for [[06 - ReentrantLock|ReentrantLocks]] in concurrency.


However, there is a problem with the previous code. `close()` can also throw an Exception **which can hide the Exception that was thrown in the `try` block**.
A better solution is to use a `try-with-resources`, which will automatically close the resources for us:
```java
// We open the resource inside the try's parenthesis
try(BufferedReader reader = Files.newBufferedReader(path)) {
	doSomething(reader);
	// ...
} // When reaching the end of the block, close() is implicitly called for us
```
> If `close()` raises an Exception, it is stored as a **suppressed exception** which can be retrieved via `getSuppressed()`.

We can initialise several variables in the `try-with-resources`, by separating them with a semicolon. In this case, files will be closed **in reverse order** (the variable declared in last will be closed first, etc)
```java
try(BufferedReader reader = Files.newBufferedReader(input); BufferedWriter writer = Files.newBufferedWriter(output)) {
	doSomething(reader, writer);
} // calls writer.close(), and then reader.close()
```


****
## Exceptions and Design by contract

As briefly explained in [[ESIPE/Semestre 3/Java/01 - Fundamentals/01 - Fundamentals#Encapsulation|this chapter]], each public method provides a usage contract (described in its Javadoc), and the implementation verifies:
- Preconditions:
    Properties of the arguments (e.g., non-null, positive).
    Properties of the object (e.g., close() has not already been called).
- Postconditions:
    Properties of the return value (e.g., non-null, positive).
    Properties of the object after the method call.
- Invariants (class invariants):
    Properties of the object that must always hold true.


Unlike unit testing, which checks if the code behaves as expected when executed, programming by contract helps detect:
- Invalid method calls (violating preconditions).
- Incorrect implementations (violating postconditions and invariants), by embedding these checks directly into the code.

In practice, preconditions are often the only ones explicitly written, as postconditions and invariants are usually handled by unit tests.

**Example of contract:**
```java
public class Stack {
	/**
	* Create a stack with a given capacity
	* @param capacity the stack capacity
	* @throws IllegalArgumentException is capacity is negative or null
	*/
	public Stack(int capacity) {/* ... */}

	/**
	* Push an object on top of the stack
	* @param object an object
	* @throws NullPointerException if the object is null
	* @throws IllegalStateException if the stack is full
	*/
	public void push(Object object) {/* ... */}

	/**
	* Remove and return the object on top of the stack
	* @return the object on top of the stack
	* @throws IllegalStateException if the stack is empty
	*/
	public Object pop() {/* ... */}
}
```

Code that goes with it:
```java
public class Stack {
	private final Object[] array;
	private int top;
	
	public Stack(int capacity) {
		if (capacity <= 0) {
			throw new IllegalArgumentException("capacity is negative");
		}
		array = new Object[capacity];
	}
	
	public void push(Object object) {
		Objects.requireNonNull(object);
		Objects.checkIndex(top, array.length);
		array[top++] = object;
	}
	
	public Object pop() {
		if (top <= 0) {
			throw new IllegalStateException("stack is empty");
		}
		Object object = array[top];
		array[top--] = null; // Garbage Collector !
		return object;
	}
}
```