[[Concurrence]]
****
**Table of Contents**
```table-of-contents
```

****
## Atomicity

Modern processors allows for atomic instructions in their assembly language
	Exchange (`XCHG`), Compare and Exchange (`CMPXCHG`)...

We can access this kind of instructions via the `java.util.concurrent.atomic` package (without the need of making our classes thread-safe with critical sections)


****
## Volatile

We call a variable volatile if it can be read or modified asynchronously by something other than the current thread of execution.
	*It provides a lightweight synchronisation mechanism, ensuring that updates to the variable are always visible to all threads*

*Note: Implementation of this concept varies depending on the language (volatile in C/C++ is different from volatile in Java)*


Volatile comes with several guarantees:
- Reads and writes are **atomic** if it concerns a **primitive value or a reference value** (not to any Object)
- When a thread writes to a `volatile` variable, the new value is **immediately visible** to other threads that subsequently read the variable.
	*This is achieved by preventing the compiler and the processor from caching the variable value in thread-local memory, forcing every read to get the latest value directly from main memory.*
- `volatile` also introduces a **"happens-before" (ordering)** relationship: Any write to a `volatile` variable happens-before every subsequent read of that same variable by any thread. This ensures that changes made to a `volatile` variable are seen in the correct order across threads.
	*This memory barrier provides the same memory visibility guarantees as a `synchronized` block, but without the mutual exclusion guarantees that comes with it...*


We can declare a `volatile` variable like so:
```java
public class Example { 
	private volatile boolean flag; // false by default
	
	public void setFlag() { 
		flag = true; // This write is visible to all threads immediately.
	}
	
	public void checkFlag() { 
		if (flag) { 
			System.out.println("Flag set"); 
		}
	} 
}
```


However, `volatile` variables does not natively allow atomic **Compound Operations** (incrementing, checking-then-updating). Its basic shape is mostly designed for flags (status updates, signalling variables in concurrent applications).


If we want to have access to those atomic compound operations (and more), we have two ways:
- **Atomic Classes** (`java.util.concurrent.atomic.AtomicInteger`...)
	**Encapsulates a volatile field (Integer here)** and provides atomic methods on it
	Since `Atomic*` is a family, there is an atomic class for every type (`AtomicBoolean` | `AtomicLong` | `AtomicDouble` ... `AtomicReference<T>`). This also applies for arrays (`AtomicIntegerArray`...)

- **VarHandle API** (`java.lang.invoke.VarHandle`)
	Since Java 9, `VarHandle` provides low-level, atomic access to fields in external classes. It **references a volatile field from an external class** and provides lower-level atomic methods on it.

*As mentioned above, both of those methods works on `volatile` variables*


****
## Atomic* Family

Let's make a simple thread-safe counter with a critical section (like we did so far):
```java
public class ThreadSafeCounter {
	private final Object lock = new Object();
	private int counter;

	public int nextValue() {
		synchronized (lock) {
			return counter++;
		}
	}
}
```

Well, this code works fine but it is kind of slow... We could make this better by using an `AtomicInteger` instead:
```java
public class ThreadSafeCounter {
	private final AtomicInteger counter = new AtomicInteger();

	public int nextValue() {
		return counter.getAndIncrement();
	}
}
```

Here, the `getAndIncrement()` method is treated as a single atomic operation by the JIT compiler.


****
## Compare and Set (CAS)
*A variant of "compare and swap", but its pretty much the same*

Unfortunately, all atomic operations aren't available on all processors. However, the foundational atomic operation across platforms **compare-and-set (CAS)** exists. 

Here is what the signature would look like in C:
```c
bool CAS(&field, expectedValue, newValue)
```

If the field's value equals `expectedValue`, it is replaced by `newValue` and returns true. It returns false otherwise.

Let's make our counter with a CAS:
```java
public class BadCounter {
	private final AtomicInteger counter = new AtomicInteger();
	
	public int nextValue() {
		for(;;) {
			int current = counter.get();
			if (counter.compareAndSet(current, counter.get() + 1) { // NO
				return current;
			}
		}
	}
}
```

This code is **wrong**. Here, we call `counter.get()` two times. It is possible for the value to change in between of those two calls !
It is mandatory to base the next value on the `current` variable's value, like so:
```java
public class ThreadSafeCounter {
	private final AtomicInteger counter = new AtomicInteger();
	
	public int nextValue() {
		for(;;) {
			int current = counter.get(); // (one) volatile read
			int newValue = current + 1;  // next based on already fetched value
			if (counter.compareAndSet(current, newValue)) { // volatile write
				return current;
			} // otherwise retry (a thread changed the value in the meantime)
		}
	}
}
```


With the introduction of lambdas, we can use the `getAndUpdate(UnaryOperator)` method to make a CAS with a loop, and pass it our next-value function:
```java
public class ThreadSafeCounter {
	private final AtomicInteger counter = new AtomicInteger();
	
	public int nextValue() {
		return counter.getAndUpdate(x -> x + 1); // cleaner code
	}
}
```


****
## VarHandle

`AtomicInteger` is easy to use, but:
- It is consuming too much memory (need to create an object for each volatile field).
- It requires a memory indirection to retrieve the value (which may cause a potential cache miss).

Since Java 9, `ava.lang.invoke.VarHandle` allows to solve those issues, but comes with a more complex API...


A `VarHandle` acts as **a pointer ("handle") on a volatile field** or array cell:
```java
// Ask for security context at this call position
Lookup lookup = MethodHandles.lookup();

// Creates a VarHandle from a security context (lookup)
VarHandle handle = lookup.findVarHandle(
	ThreadSafeCounter.class, // Class containing the field
	"counter",               // Field name
	int.class);              // field type
```


Let's make our thread-safe counter with a `VarHandle`:
```java
public class ThreadSafeCounter {
	private volatile int counter;
	private static final VarHandle COUNTER_REF;

	/*
	Static initializer block: used to initialize static fields or perform setup
	tasks when the class is first loaded.
	*/
	static {
		Lookup lookup =	MethodHandles.lookup();
		try {
			// Allows direct access to counter field for atomic operations
			COUNTER_REF = lookup.findVarHandle(
				ThreadSafeCounter.class, 
				"counter", 
				int.class);
		} catch (NoSuchFieldException | IllegalAccessException e) {
			/*
			NoSuchFieldException can be thrown if specified field does not exist
			IllegalAccessException can be thrown if there is an access issue
			*/
			throw new AssertionError(e);
		}
	}

	public int nextValue() {
	    for(;;) {
			int current = this.counter; // volatile read
			if (COUNTER_REF.compareAndSet(this, current, current + 1)) {
				return current;
			} // otherwise retry
		}
	}
}
```


This code works, but we can optimise it. There is an equivalent to the `getAndIncrement()` method for `AtomicInteger`: the static `VarHandle.getAndAdd()` method:
```java
public class ThreadSafeCounter {
	private volatile int counter;
	private static final VarHandle COUNTER_REF;

	static {
		Lookup lookup =	MethodHandles.lookup();
		try {
			COUNTER_REF = lookup.findVarHandle(
				ThreadSafeCounter.class, 
				"counter", 
				int.class);
		} catch (NoSuchFieldException | IllegalAccessException e) {
			throw new AssertionError(e);
		}
	}

	/*
	VarHandle does not use generics (diamond syntax) :(
	We are required to make an ugly cast to indicate the return type
	*/
	public int nextValue() {
	    return (int) COUNTER_REF.getAndAdd(this, 1);
	}

	// If our counter was working on longs:
	// public long nextValue() {
	//     return (long) COUNTER_REF.getAndAdd(this, 1);
	// }
}
```


Our counter is now pretty performant, only one small detail remains. By default, the `VarHandle` API is doing **boxing** (int -> Integer ...)
	*This is one of the biggest problem with Java by the way, as this poor design choice destroys performance (this behaviour was fixed in Kotlin)*

We will prevent the autoboxing feature on our `VarHandle` like so:
```java
// ...

private static final VarHandle COUNTER_REF;

static {
	Lookup lookup =	MethodHandles.lookup();
	try {
		COUNTER_REF = lookup.findVarHandle(Class.class, "counter", int.class)
			.withInvokeExactBehavior(); // Here
	} catch (NoSuchFieldException | IllegalAccessException e) {
		throw new AssertionError(e);
	}
}

// ...
```


****
## Data Structures

In fact, **almost all concurrent data structures** (present in the `ava.util.concurrent` package) natively included in Java **doesn't use `synchronized` blocks.**
We either use a `ReentrantLock`, or go with a **lock-free implementation** (`volatile` or Atomic operations).

Let's try to make this basic linked list concurrent:
```java
public final class NotConcLinkedList {
	private record Entry(Object value, Entry entry) {}
	private Entry head;
	
	public void add(Object value) {
		this.head = new Entry(value, this.head);
	}
	// ...
}
```

This implementation works with `AtomicReference`:
```java
public class LockFreeLinkedList {
	private record Entry(Object value, Entry next) {}
	private final AtomicReference<Entry> reference = new AtomicReference<>();

	public void add(Object value) {
		for(;;) {
			Entry head = this.reference.get();
			Entry entry = new Entry(value, head);
			if (this.reference.compareAndSet(head, entry)) {
				return;
			}
		}
	}
}
```

This implementation works with `VarHandle`:
```java
public class LockFreeLinkedList {
	private record Entry(Object value, Entry next) {}
	private volatile Entry head;
	private static final VarHandle COUNTER_REF;

	static {
		Lookup lookup =	MethodHandles.lookup();
		try {
			COUNTER_REF = lookup.findVarHandle(
				LockFreeLinkedList.class, 
				"head",
				Entry.class)
				.withInvokeExactBehavior();
		} catch (NoSuchFieldException | IllegalAccessException e) {
			throw new AssertionError(e);
		}
	}

	public void add(Object value) {
		for(;;) {
			Entry head = this.head; // volatile read
			Entry entry = new Entry(value, head);
			if (HEADER_REF.compareAndSet(this, head, entry)) { // volatile write
				return;
			}
		}
	}
}
```
