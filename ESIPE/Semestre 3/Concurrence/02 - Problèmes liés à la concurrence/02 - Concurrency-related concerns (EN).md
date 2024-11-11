[[Concurrence]]
****

We have a big issue: Threads can be **de-scheduled right in the middle of a set of instructions, but even worse, right in the middle of a single instruction/operation**. 

This is due to the fact that most operations are **non-atomic** (or "non-synchronised"). 

We observed in the previous lab that this issue never happened with `System.out.println` as this method uses a **lock** (which is released once the print operation is finished).
	*Details in the [[03 - Synchronize and Thread-Safe (EN)|next lesson]]*


****
## Memory and Non-Atomicity

Recall that threads shares a same memory space: the heap. 
The usage of non-atomic operations brings major issues: 
- Interlaced access to memory (read, write) 
- Code optimisation by the JIT
- CPU using cache

We call a **data race** a situation where **several threads can access an identical memory zone**:
```java
public class Counter {
	private int value;

	public void addALot() {
		for (int i = 0; i < 10_000; i++) {
			this.value++;
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

This code is obviously wrong and will never print the expected result. Here, the data race occurs on the `value` field of the `counter` object.

In a perfect world, `value` should be 20000. But it will — almost — never occur.
	*In theory, we could even end up with `value` of 2. Details in next lab.*
