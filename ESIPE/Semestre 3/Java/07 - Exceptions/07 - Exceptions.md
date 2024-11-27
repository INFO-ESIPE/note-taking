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
- **Unchecked:** Exceptions that mostly happens due to programming errors

So, any Exception which inherits from `Throwable` must be checked, except the ones inheriting from `RuntimeException` and `Error`.


****
## Exceptions and I/O


****
## Exceptions and Design by contract
