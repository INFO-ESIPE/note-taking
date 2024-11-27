[[Concurrence]]
****
**Table of Contents**
```table-of-contents
```

****
## Mutex

Non-atomicity of instructions is a major issue, as our threads might be de-scheduled in the middle of it, allowing other threads to see data in an incoherent state.

We need a solution allowing to include a **synchronisation** logic.
This solution is a **critical section** (lock). 

The simplest and most common implementation of a critical section is the **mutex (MUTual EXclusion)**.
A critical section is a block of code that accesses a shared resource and can’t be executed by more than one thread at the same time. This section is attached to the lock.
	*If a thread is inside one of the critical sections binded to a lock, no other thread can enter any critical section binded to the same lock.*

When used correctly, this ensures that several threads can not access some code (and the data manipulation that goes with it) at the same time.
The ultimate purpose is to **prevent thread 2 from attempting to read a value that thread 1 is currently modifying or reading, until the overall operation he carries is fully done**


However, **the lock does not guarantee that the thread wont be de-scheduled right in the middle of the critical section**. It just prevents the reading and execution of fragments of code by several threads, but does not guarantee the writing on RAM. 
To solve this issue, it will be mandatory to **apply the lock on each reading/writing action of the class**.

*For our `count` from last time:*
```java
public class Counter {
	private int value;

	public void addALot() {
		for (int i = 0; i < 10_000; i++) {
			// Begin critical section
			this.value++;
			// End critical section
		}
	}

	public int value() {
		return this.value;
	}

	public static void main(String[] args) throws InterruptedException {
		var counter = new Counter();

		var thread1 = Thread.ofPlatform().start(counter::addALot);
		var thread2 = Thread.ofPlatform().start(counter::addALot);

		System.out.println("Final value: " + counter.value());
	}
}
```


****
## Synchronise block

The `synchronized` block is a memory boundary that prevents concurrent execution of a block of code, as it always results in altering data in an incoherent manner.

The thread entering the block **acquires a token (the lock)** associated to the block, and **releases the token at the end of the block** (when it is about to leave it). 

**Only one thread at a time can carry this token**. If the thread carrying the token is de-scheduled in the middle of the `synchronized` block, **he keeps it** and no other thread can access any `synchronized` block binded to this lock (as no other thread can acquire the token).
	*Try to visualise it as a lock in real life. The first person to enter the lock-protected room takes the key, and keeps it. As long as he doesn't leaves the room, he keeps the key. Other people willing to enter the room can't, until he leaves and give back the key.*


**The synchronise mechanism also prevents the JIT to swap lines of code** (when he optimises the code) **outside of the critical section**, for obvious reason. 
Instructions can be swapped in the section itself, but it will never escape it.
```java
// Normal code
var i = 20;
var j = 30;
var k = j + i;

// What the JIT can do if it judges it more optimized and that it does not alternate the behaviour of a SEQUENTIALLY EXECUTED CODE (so, not concurrent)
var j = 30;
var i = 20;
var k = j + i;
```


The synchronisation mechanism does not guarantee the correct behaviour of the concurrent code outside of the critical section itself. If we execute non-atomic and non-synchronised operations from outside (e.g,
```java
System.out.println(user.nom() + " " + user.prenom())
```
), then the mechanism loses all it's purpose and the code will potentially opens on concurrency issues. 
So it is always important — even if the class is thread-safe and well-conceived — to **respect concurrency principles outside of the class**.


So, we see that this synchronise mechanism does not prevent de-scheduling in any way — and by doing so, an incoherent state of data — but it will definitely prevent other threads from accessing this incoherent state. 
	*Which means, it does not really matter that our data is incoherent during a critical section, as only the current thread can read/write it. For other threads, the data will only be accessible when everything is done.
	We have the illusion that the state is never incoherent (which is false, a thread can still be interrupted in the middle of a non-atomic operation), but since no one else can see it, it's fine.*


****
## Mutex - Implementation and usage in Java

As we explained, a `synchronize` block is binded to a lock. In our class, we initialise an `Object` that will serve us as the lock (also called **"monitor"**):
```java
public class Counter {
	private int value;
	private final Object lock = new Object();

	/* 
	Note that here, it is more performant to put the synchronize inside the loop rather than outside. If a thread is de-scheduled after the end of a loop, another thread can get the lock and increase the counter on it's side without causing any data race. If the synchronize was outside of the loop, the code would only be accessible if the current thread finished the 10000 loops
	*/
	public void addALot() {
		for (int i = 0; i < 10_000; i++) {
			synchronized(lock) { // Begin critical section
				this.value++;
			} // End critical section
		}
	}
}
```

The chosen lock **must respect three conditions**: 
- **The lock must not be accessible to other threads** and must then respect several encapsulation principles: 
    - Monitor field must be **private** as it mustn't be retrieved from outside (obviously, don't even start doing retarded things like a `getter` or `setter` for the lock, it **SHOULD ONLY BE ACCESSIBLE WHEN A THREAD ENTERS A CRITICAL SECTION, NOTHING MORE**)
    - Lock must be **final**, as it **must be initialised in the constructor and never change afterwards** 
    - As mentioned above, **never provide an accessor to the lock**
- It must be an **object**, so no primitives (int, long...) nor null 
- The chosen object **mustn't implement interning** (String literals). More details [[03 - Synchronize and Thread-Safe (EN)#Interning|here]] 


Locks in java are called **"re-entrant"**, a **same thread can acquire several time the same lock**:
```java
public class Foo {
	private int value;
	private final Object lock = new Object();

	public void setValue(int value) {
		synchronized(lock) {
			this.value++;
		}
	}

	public void reset() {
		synchronized(lock) {
			setValue(0);
		}
	}
}
```


****
## Interning

Interning is an optimisation avoiding useless allocations for objects having the same value. 
	*If there are fifteen "Bob" Strings, the JIT only allocates memory for the first one, the forty-nine others will simply point to a **reference/canonical representation** (similar to a C pointer) to the very first "Bob" String, so we don't have to allocate several times some memory space to store the same thing...*

This is what it looks like under the hood: 
	There is an `intern()` method on the String class which operates the aforementioned action. There is a pool of Strings inside the program which is maintained by the `String` class. This collection is initially empty.
	When the `intern()` method is called, the class looks if it already exists in the pool (via the `equals()` methods, obviously not `==` huh...)
	If it is present, we return the original string present in the pool, and do not allocate anything. Otherwise, we add this new string to the pool and return the reference to it.


By default, JIT is doing Interning on the `String`. This is done in an implicit way, which is why we never use the dedicated method to do it, as the JIT does it for us.


****
## "Thread-Safe" Class

A class is thread-safe when it **can be used by several threads without having data race nor incoherent states** 
	*kind of an abstract definition...*

As mentioned earlier, thread-safe guarantees the good conception of the class itself, not it's correct usage from outside. 

Most native mutable Java classes aren't thread safe (`HashMap`, `ArrayList`...). This will radically change the way we conceive our code. 
The only native thread-safe classes are the one in the `java.util.concurrent` package. 
	*Note: a record (or any immutable class) is, by definition, thread-safe...*

