[[Concurrence]]
****

For now, we have no way to set up a "rendezvous" mechanism

```java
public class Holder {
	private int value;

	public static void main(String[] args) {
		var holder = new Holder();
		Thread.ofPlatform().start(() -> {
			holder.value = 42;
		});
		
		System.out.println(holder.value == 42 ? "OK" : "KO");
	}
}
```

We would like, in this example, to only print when the data is set. This program will sometimes print `OK`, and sometimes `KO` (not the expected behaviour).


A bad idea would be to proceed like this:
```java
public class Holder {
	private int value;
	private boolean done;

	public static void main(String[] args) {
		var holder = new Holder();
		Thread.ofPlatform().start(() -> {
			holder.value = 42;
			holder.done = true;
		});
		while (!holder.done) {
			// wait... do nothing ...
		}
		System.out.println(holder.value == 42 ? "OK" : "KO");
	}
}
```

This way, the main thread only moves forward if the holder is done applying the value. There is an obvious issue there: **Busy Waiting**
	*The thread continuously checks if the boolean is true, which consumes heavy resources on the CPU*
There is actually another issue there: **Instruction Reordering**
	*The Just-In-Time compiler (JIT), and even the CPU, can reorder instructions on their own to optimise the program. This means we could obtain a code like this:*
```java
Thread.ofPlatform().start(() -> {
	holder.done = true;
	holder.value = 42;
});
```
> It is now possible for the main thread to consider the overall operation done (boolean set to true), even though the value hasn't been specified yet...


We need to avoid both of those behaviour...
```java
public class Holder {
	private int value;
	private boolean done;
	private final Object lock = new Object();

	public static void main(String[] args) {
		var holder = new Holder();
		Thread.ofPlatform().start(() -> {
			synchronized (holder.lock) {
				holder.value = 42;
				holder.done = true;
			}
		});
		
		for (;;) { // busy waiting still...
			synchronized (holder.lock) {
				if (holder.done) {
					break;
				}
			}
		}

		/* Mandatory !! Without it, there is no guarantee that the value variable is read back into memory. As another thread is doing the writing, it could very well simply be read back into the cache and not into RAM. */
		synchronized (holder.lock) {
			System.out.println(holder.value == 42 ? "OK" : "KO");
		}
	}
}
```


 Actually, the best way of doing it is through a get/display method inside the class, which implements the `synchronize` itself :

```java
public class Holder {
	private int value;
	private boolean done;
	private final Object lock = new Object();

	private void init() {
		synchronized (lock) {
			value = 42;
			done = true;
		}
	}

	private void display() {
		for (;;) { // busy waiting still...
			synchronized (lock) { 
				if (done) { 
					break; 
				} 
			}
		}
		synchronized (lock) { 
			System.out.println(value == 42 ? "OK" : "KO"); 
		}
	}

	public static void main(String[] args) {
		var holder = new Holder();
		Thread.ofPlatform().start(holder::init);
		holder.display();
	}
}
```


This is better, but this does not resolve our busy waiting issue at all...
We need the following mechanism to solve our issue:
- Can pause a thread (de-scheduled)
- Wake up a thread (re-schedule)
- Works well in critical sections (`synchronize`-compatible)

In java, this behaviour is offered to us by the `wait()` and `notify()` pair !


****
## `notify()` and `wait()`

Those instance functions are **called on the lock**. 
```java
lock.notify();

while (!done) {  
	lock.wait();
}
```

`wait()`:
1. Puts the current thread asleep
2. Gives back its token for the `lock`
3. When the thread is waken up later (by a notify signal), it takes back the `lock` token (if available). The thread has retrieved its execution flow.

`notify()`: Wakes up **one random thread that is AWAITING ON THE LOCK**
	*If there are several, the scheduler picks one randomly. If there are none, the notify signal does nothing and is lost (careful...)*
	
`notifyAll()`: Wakes up **all the threads AWAITING ON THE LOCK**. This should be used as few as possible, especially when a simple `notify()` can be used.

**Those methods must ALWAYS be called inside a `synchronized` block !!!!!**


A thread can only be woken up in three scenarios :
- Received a `notify`/`notifyAll` signal (as we have seen just right now)
- Another thread sent an **interruption signal** with `interrupt()` (more on that in a future lesson)
- **Spurious wakeup**. Those usually happens when the wrong thread is awaken, or when the OS is busy and decides to wake up all the threads.


****
## Code with the signal pair

Let's fix the busy waiting problem in our code:

```java
public class Holder {
	private int value; 
	private boolean done; 
	private final Object lock = new Object();

	private void init() {
		synchronized (lock) {
			value = 42;
			done = true;
			lock.notify();
		}
	}

	private void display() throws InterruptedException {
		synchronized (lock) {
			while (!done) {  
				lock.wait(); // no more busy-waiting !
			}
			System.out.println(value == 42 ? "OK" : "KO"); 
		}
	}

	public static void main(String[] args) throws InterruptedException {
		var holder = new Holder();
		Thread.ofPlatform().start(holder::init);
		holder.display();
	}
}
```


**Why do we keep the boolean ?**

If the thread that sets the data notifies before the main thread reaches the `wait()`, the signal is lost, and the main thread is blocked here forever.
	*It is good practice to use a **condition** (more on that with ReentrantLocks) to know how to deal with the wait. This condition can be a boolean, or something of that kind... **ALWAYS IN THE `synchronized` BLOCK THOUGH***


**Why `wait()` in a `while` and not an `if` ?**

When we are waken up, we always need to make sure the condition is satisfied. 
	*In the scenario of an `if`, the threads reach the condition, see that it is not satisfied, and sleeps. When it is awaken, it leaves the `if` and executes the instructions ahead. However, we are not sure that a **spurious wakeup** happened, the condition is potentially still not met, which means the thread should go back to sleep and wait for another wakeup signal.*

