[[Concurrence]]
****
**Table of Contents**
```table-of-contents
```

****
## Reminder

In Java, there are only two ways of getting out of a function:
- `return` (implicit or explicit)
- Raised `Exception`

There are two kinds of `Exceptions`:
- **Unchecked** (usually errors from the programmer)
	*`NullPointerException`, `ArrayIndexOutOfBoundsException` ...*
- **Checked** (usually due to an exterior action)
	*`IOException`, `InterruptedException`*


****
## Stop a Thread

It is not possible to force a thread to stop (kill) in Java. The only way we can achieve this is by **asking him to do so via a signal** 
	*It is some kind of "cooperative" approach. 
	An interrupt signal means that we asked for the thread to stop, however, he can still do whatever he wants. The code used by the thread must know how to deal with `InterruptedException`
	**DOING NOTHING (OR CALLING `printStackTrace`) IS NEVER THE GOOD SOLUTION***

Thread A can send an interruption signal to thread B like so:
```java
threadB.interrupt(); // sends interruption signal
```


Each thread has an **interruption status flag**, which is nothing more than a boolean property in the object.
We can check this status in two ways:
- Static Method: `Thread.interrupted()`
	*This methods check if the flag is set for current thread (executing the instruction), but it **SETS THE FLAG BACK TO FALSE**
	In other words, if this method were to be called twice in succession, the second call would return false (unless the current thread were interrupted again, after the first call had cleared its interrupted status and before the second call had examined it).
		The naming of this method was poorly chosen, it should have been named `Thread.interruptedAndClear()` as it reposition the flag to false after it checked its value*
```java
var thread = Thread.ofPlatform().start(() -> {
	for (var l = 0; l < 1_000_000_000L && !Thread.interrupted(); l++) {
		// long calculation
	}
});

// ...

thread.interrupt();
```

- Instance method : `myThread.isInterrupted()`


**Blocking method**

A method is blocking if it puts the current thread asleep. This includes:
- IO Syscalls
- `Thread.sleep()`
- `wait()`, `await()`

When a thread receives an interruption signal, two scenarios can happen:
- **The thread is not in a blocking function**: The flag is set to true
- **The thread is in a blocking function**: A Checked `InterruptedException` is raised (and the flag is reset to false)

As we have said earlier, Checked exceptions must be handled. We can do so in several ways
```java
var thread = Thread.ofPlatform().start(() -> {
	for(;;){
	 try {
		Thread.sleep(5_000); // blocking
	 } catch (InterruptedException e) {
		return; // exit, which ends the thread
	 }
	}
});

// ...

thread.interrupt();
```


**Note:** Some blocking IO functions raises `IOInterruptedException` instead of `InterruptedException`, mind the signature of methods to avoid bad surprises
	*When one of those are raised, it is mandatory to stop the thread (or for a server, to close the connection with the client)*


****
## Entering a critical section

Entering a `synchronized` is a blocking action if another thread holds the token, however, it does not raise an `InterruptedException`.
If we want to implement this interruption mechanism for the wait, we need to use a `ReentrantLock` with a `lockInterruptibly()` as explained [[06 - ReentrantLock#Condition|here]]

