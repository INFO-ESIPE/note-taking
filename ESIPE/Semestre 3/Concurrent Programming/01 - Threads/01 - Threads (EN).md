[[Concurrence]]
****
**Table of Contents**
```table-of-contents
```

****
## Concurrency ?

Concurrent programming is a paradigm that allows for different fragments of code to be executed at the same time. This mostly is applied in two scenarios: 
- Blocking I/O calls (Database queries, API calls...)
	*Most frequent case for software development, and the one we are interested about in this class*
- Massive calculations requiring a complete use of the CPU
	*We'd rather use OpenMP or MPI for this*


****
## Threads and processes

When a program is executed, a process is created by the OS. This process has its isolated virtual address space. It is composed of **execution thread**.

A thread is **a lightweight unit of execution within a process**, allowing for concurrent operations.
	*A program has at least one thread, but it can have several (hence the purpose of this class...)*

Each thread **possesses its own stack** (managing function calls, parameters, local variables...), but **shares the same heap memory, global data, text segment (code) and data segment**.
- In Java, stack contains local variables and function parameters. 
- In Java, the heap contains objects's properties.

![[threads.png]]


The **scheduler** is an OS-managed entity **deciding on threads' execution order and alive time**. We will use the `java.lang.Thread` class to manage threads.

Pretty much all programming languages shares a similar way of implementing concurrency. No language really craft or innovate concurrency concepts, they just use what the OS and CPU vendors provides.

A `Thread` takes no parameter and returns nothing, it's a [[02 - Lambdas#`Runnable`|Runnable]] .
```java
@FunctionalInterface
public interface Runnable {
	/* public abstract */ void run();
}
```

We initialise and start a thread like so:
```java
var p = "Hello World!"

// Before Java 19 (to avoid)
Runnable runnable = () -> {              // Runnable the thread will execute
	System.out.println(p);
};
Thread preThread = new Thread(runnable); //  Creates thread
preThread.start();                       // Starts the thread 


// Java 19 and beyound (to use)
Thread thread = Thread.ofPlatform().start(() -> { // Create and launch
	System.out.println(p);
});

Thread unstartedThread = Thread.ofPlatform().unstarted(() -> { // Launch it later
	System.out.println(p);
});
unstartedThread.start();
```


The `Thread` object is created via the `ofPlatform()` method factory.
The **creation of the object does not start the thread**. To do so, it is required to call the `start()` method on it, which will do the following:
- Allocate a new stack
- Ask the OS for the creation of a new thread (platform thread) 
- The thread executes the `run()` method of the `Runnable` Functional Interface 
- **Once the execution of the method ends, the thread dies**

Even if the thread is dead, the object itself only "dies" once it has been freed by the **garbage collector**.

**Example:**
```java
public class Application {
	private static void myRun() {
		System.out.println("Hello myRun");
	}

	public static void main(String[] args) {
		Runnable runnable = () -> {
			System.out.println("Hello main");
		};
		Thread thread1 = Thread.ofPlatform().start(Application::myRun);
		Thread thread2 = Thread.ofPlatform().start(runnable);
	}
}
```


There is **no father-son relationship** between threads (unlike `fork()` in C or system programming overall). 
	*If we launch a new thread from the main one, the main thread can die before the concurrent one without stopping the entire program. The main thread is just a thread like the others, it's only the backbone of the process at the beginning as we do need a thread to start a process and do basic actions on it*

**Threads do not communicate with each other**, they can not re-launch each other directly (nor re-launch themselves, in fact).


****
## Scheduler

In order to properly balance the various execution threads on all the CPU cores, the scheduler is here to find empty cores to assign a thread to.
	*A thread runs on a full core*

Since an OS runs way more threads than cores available, it is the scheduler job to put some threads at rest, and launch others in the meantime. Execution of threads rotates, so everyone can be in the foreground for a moment.


A thread occupies it's designated core: 
- Until the next blocking call (I/O) 
- Until a specific amount of time where he is "**de-scheduled**"

The scheduler is supposed to give the same amount of CPU lifetime to all threads, it is **"fair"**.
	*This is only in theory. In reality, it is indicated that the scheduler is fair as it gives `time to all threads with no preferences`, but it does not indicate that `each thread receives the same amount of lifetime`. This up time is too variable to mention real "fairness".*

As all of this is managed by the OS' scheduler, **we do not control anything**, neither when the threads are scheduled or de-scheduled, nor their lifespan when they are up.


****
## Virtual and Platform Threads

There is, in Java (and other languages too, even though they use different naming for a similar concept), two types of threads:
- **Platform Thread**: Scheduled entirely by the OS, which requires some kind of context commutation (context switch: userland / kernel)

- **Virtual Thread (experimental since java 19)**: Managed by the JVM's internal scheduler, which reduces latency issues due to the context switch.
	*Those Threads consumes way fewer resources. A single program can launch thousands and thousands of virtual threads without any performance issue, which is not possible with `ofPlatform` threads.*

Both kinds are managed under a common `Thread` class. If we want to make a virtual thread, we process like so:
```java
Runnable runnable = ...;
Thread thread = Thread.ofVirtual().start(runnable); // Instead of ".ofPlatform()"
```


The JVM scheduler creates as much platform threads as there are CPU cores, and schedules all the virtual threads on those. We still have no control over the OS' scheduler, but this abstraction gives more performance.
	*Our lack of control on the scheduler is the main characteristic (and difficulty) of concurrent programming.* 

A virtual thread occupies a platform thread: 
- Until the next blocking action (I/O)
- If a virtual thread is not doing a blocking action, then a new platform thread is created so the scheduler remains fair


As the virtual thread feature remains experimental and has still a lot of unpatched issues, we will only use `ofPlatform` for this class.


****
## Properties of a thread

We can name it. We only do this for **debug purposes**, and never to pass values in between threads like a mongoloid:
```java
var thread = Thread.ofPlatform().name("Listener thread").start(runnable);
```

We can **retrieve the Thread executing the current code**:
```java
Thread.ofPlatform().name("Listener thread").start(() -> {
	var groot = Thread.currentThread(); // here
	System.out.println("I am " + groot);
	System.out.println("My name is " + groot.getName());
});
```


As we don't manage the execution order, we **cannot predict in which order the names bellow will be printed**:
```java
public static void main(String[] args) {
	Runnable runnable = () -> {
		System.out.println(Thread.currentThread().getName());
	};

	Thread.ofPlatform().name("Thread 1").start(runnable);
	Thread.ofPlatform().name("Thread 2").start(runnable);

	System.out.println(Thread.currentThread().getName());
}
```

*We can't know which of the three threads will be executed first, and print...*


****
## Death :(

We cannot kill or restart a thread (unlike a process), it is however possible to **gently ask him to end his life, but there is no guarantee** it will do it.
	*This is a more advanced topic covered [[07 - Interruptions and Exceptions|here]], the code needs to be conceived in a way that it takes into account those "suicide requests", and deal with them correctly*

A thread **dies when the run() is over**: 
- We reached the end of the function
- We reached an early `return` instruction
- An `Exception` was raised

The JVM only stops if all threads are dead, besides **daemon threads**. The main thread can perish before other threads, it does not matter nor close the JVM.
	*A daemon thread acts like a background service. It runs like other threads but, if no non-daemon threads are alive, it will end its existence so it does not prevent the JVM from closing.*

It is possible for a thread to **wait for another thread's death**, thus giving room for an execution order on the code:
```java
// Mind the "throws" in the method signature !
public static void main(String[] args) throws InterruptedException {
	var thread1 = Thread.ofPlatform().start(() -> {
		System.out.println("Thread 1");
	});

	var thread2 = Thread.ofPlatform().start(() -> {
		System.out.println("Thread 2");
	});

	thread1.join(); // Main thread waits for thread1 to die before it continues
	thread2.join(); // Main thread now waits for thread2 to die

	System.out.println("Thread main"); // Now that both threads died, we can print
}
```

Calling `thread.join()` can raise an `InterruptedException`, which is triggered when we ask for a thread to stop while he is in a blocking action. This topic is covered in more details [[04 - Signals|here]] and [[07 - Interruptions and Exceptions|here]].
	*As `.join()` is a blocking action, it can throws the aforementioned exception*

The `Thread.sleep()` method allows to **pause the current thread for `n` milliseconds**. Just like `join()`, it is a blocking method and can throw an `InterruptedException`.
In this case, we need to **manage the Exception with a try-catch and an `AssertionError`** :
```java
Thread.ofPlatform().start(() -> {
	try {
		Thread.sleep(5000); // 5 sec
	} catch (InterruptedException e) {
		throw new AssertionError(e);
	}
	
	System.out.println("Done");
});
```

