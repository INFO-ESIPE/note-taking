[[Concurrence]]
****
Note: Some content detailed here is already familiar to us so far
****

Java follows the **Write Once Run Anywhere (WORA)** paradigm.
	*Java code can run on any OS/architecture, as long as this device can run the JVM*

But every OS comes with its own rules and choices of architecture, how can Java guarantee the same flow of execution on every device ?

Java has its own memory model that abstracts a machine's own memory model, thus allowing uniform **execution and visibility guarantees** across all platforms.


****
## Happens-before

This memory model establishes the **happens-before** principle: an operation can see the state of a previous operation

**Within a single thread:** When a thread reads a variable it previously wrote to, it will always see the last value it assigned (**visibility guarantee**).
```java
Point p = // ...
p.x = 1;
p.x = 3;
int a = p.x; // <==> a = 3
```


**Across Multiple Threads**: There is **no visibility guarantee** for fields accessed by several threads, unless synchronisation mechanisms are used (`final`, `volatile`, `synchronize`...)
```java
// Thread 1               Thread 2
p.x = 1;                  p.x = 3;
p.x ? /* 1 or 3 */        p.x ? /* 3 or 1 */
```


`thread.start()` guarantees that all previous writings are visible to the starting thread:
```java
Point p = new Point();
Runnable runnable = () -> {
	System.out.println(p); // guarantee that p is non null
};

Thread t = new Thread(runnable);
t.start();
```

`thread.join()` guarantees that writings made by the joined thread will be visible by the current thread:
```java
class MyRunnable implements Runnable {
	private int result;
	
	public void run() {
		result = 3;
	}
});

MyRunnable runnable = new MyRunnable();
Thread t = new Thread(runnable);
// ...
t.join();
System.out.println(runnable.result); // 3
```


****
## Publication and Initialisation Safety

A field that is declared `final` is visible by all threads once we leave the constructor:
```java
class A {
	private static A a;
	private final int value;
	
	A(int value) {
		this.value = value;
	}
	
	public static void main(String[] args) {
		new Thread(() -> {
			if (a != null) {
				System.out.println(a.value); // 7
			}
		}).start();
		a = new A(7);
	}
}
```

If we don't specify that `value` is `final`, our code is incorrect as we can see the default value.
	*The `println` can either print 7, or 0 (default value for an int until it is assigned a value)*


Fields initialised in a `static` block are visible to all threads after the class has been loaded. 
	*This makes static blocks useful for initialising singleton instances*


`volatile` has three effects:
- Writing to a `volatile` variable ensures all previous writes are visible to other threads that read this variable afterwards.
- Reading a `volatile` field forces all subsequent reads of other variables to refresh their values from main memory.
- `volatile` also ensures atomicity for 64-bit values like `long` and `double`, even on 32-bit machines.
```java
class A {
	private int value;
	private volatile boolean done;
	
	public void init(int value) {
		this.value = value;
		this.done = true; // volatile write, so value is written in RAM
	}
	
	public static void main(String[] args) {
		A a = new A();
		Thread t = new Thread(() -> {
			a.init(9);
		}).start();
		if (a.done) { // volatile read, cache invalidation
			System.out.println(a.value); // must be re-loaded from RAM, so 9
		}
	}
}
```


`synchronize` has two effects:
- Entering a `synchronized` block reloads values from main memory
- Exiting a `synchronized` block writes all changes back to main memory
	*This guarantees visibility of changes to any thread that later synchronises on the same monitor.*
```java
class A {
	private int value; // no volatile needed
	private boolean done; // no volatile needed
	private final Object lock = new Object(); // if not final, publication problem!

	public void init(int value) {
		synchronized(lock) {
			this.value = value;
			this.done = true;
		} // value and true are written in RAM
	}

	public static void main(String[] args) {
		A a = new A();
		Thread t = new Thread(() -> {
			a.init(9);
		}).start();

		// Note: this would be better to have a "done" method that does the synchronize for us
		synchronized(a.lock) { 
			if (a.done) { // done and value are re-loaded from RAM
				System.out.println(a.value); // 9
			}
		}
	}
}
```


****
## Example: Singleton

Singleton is a design pattern that guarantees the existence of only one instance of a class
	*This is commonly use for databases*

This works well since **java classes are lazy** (loaded and initialised only when first accessed, not before), unlike C++ classes for instance...
```java
public class DB {
	private static final DB INSTANCE = new DB(); // Create the only instance here
	// Other fields ...

	private DB() { // PRIVATE constructor, can not instantiate outside
		// init fields ...
	}
	
	// Only way to get DB is through the already existing instance
	public static DB getSingleton() { 
		return INSTANCE;
	}
}
```


Since java classes are lazy, and since our instance is final (initialised when first accessed), this works fine. However, what if the INSTANCE is not final ? Well, the code is not thread safe:
```java
public class DB {
	private static DB INSTANCE; // Not final!
	// Other fields ...

	private DB() {
		// ...
	}
	
	public static DB getSingleton() {
		if (INSTANCE == null) {
			INSTANCE = new DB();
		}
		return INSTANCE;
	}
}
```

We can fix this with a `synchronize` block, like we have done several times already:
```java
public class DB {
	private static DB INSTANCE; // Not final!
	private static final Object lock = new Object(); 
	// Other fields ...

	private DB() {
		// ...
	}
	
	public static DB getSingleton() {
		synchronized(lock) {
			if (INSTANCE == null) {
				INSTANCE = new DB();
			}
			return INSTANCE;
		}
	}
}
```
Note that putting the `synchronized` inside the if condition will NOT WORK.
