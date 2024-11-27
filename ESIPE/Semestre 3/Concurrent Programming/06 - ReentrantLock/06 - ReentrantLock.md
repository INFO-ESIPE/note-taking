[[Concurrence]]
****
**Table of Contents**
```table-of-contents
```

****
## Alternative to `synchronize`

Since Java 5, we can also use the `ReentrantLock` class (which implements the `Thread` interface) to define critical sections.
- Real object with methods and more flexibility
- Several signals on a same lock (**Conditions**)

This is more advanced than the simple `synchronize`. 


****
## ReentrantLock

Just like `synchronize`, it follows a mutex concept.
- `lock()` method allows to take the token
- `unlock()` method allows to drop the token

A `lock()` call **ALWAYS IMPLIES AN `unlock()` CALL !** 
	*The code in between those two calls is the critical section*


A first naive approach would be to do something like this:
```java
public class StupidLockDemo {
	private final ReentrantLock lock = new ReentrantLock();

	public void increment() {
		lock.lock();
			
		// code 
		
		lock.unlock();
	}
}
```

But this is wrong and dangerous. Java code is susceptible to raise a lot of Exceptions. If an exception occurs before the `unlock()` method is reached, we never give our token back... 

For this reason, we always need surround our code (and `unlock()`) in a `try/finally`:
```java
public class SwagLockDemo {
	private final ReentrantLock lock = new ReentrantLock();

	public void increment() {
		lock.lock();
		try {
			// code 
		} finally {
			lock.unlock();
		}
	}
}
```


**Fairness**

There are two versions of `ReentrantLock` :
- **Fair** - If several threads are waiting on a lock, priority goes to the one that has awaited for the longest time (this has nothing to do with thread priority, the longest waiter always wins here)
```java
private final ReentrantLock lock = new ReentrantLock(true); // with param
```

- **Unfair** (default) - If several threads are waiting on a lock, a random one gets it
```java
private final ReentrantLock lock = new ReentrantLock();
private final ReentrantLock lock = new ReentrantLock(false); // works aswell
```

*The fair version is slower...*


**Methods**

ReentrantLock comes with a few useful methods that a simple `synchronize` does not offer :
- `tryLock()`: Acquire the lock (just like `lock()` method) but returns a `false` boolean value if it could not acquire it (instead of blocking). **This operation is atomic**
- `lockInterruptibly() throws InterruptedException`: Acquire the lock (just like `lock()` method), but the operation be interrupted if the thread is locked while waiting for the token (if another thread holds it in the meantime)


****
## Condition

**Caution:** We can not use `lock.wait()` nor `lock.notify()`/`lock.notifyAll()` as those suppose we are in a `synchronized` block, which is not the case

We pair our `ReentrantLock` with a `Condition` object that it can provides us:
```java
private final ReentrantLock lock = new ReentrantLock();
private final Condition condition = lock.newCondition();
```


We use the following methods on the condition:
- `condition.await()` instead of `wait()` (this is tricky, always be careful to call await as most native classes are binded with a `wait` method that does not act as we want at all)
- `condition.signal()` instead of `notify()`
- `condition.signalAll()` instead of `notifyAll()`

*Those methods works exactly the same as their `synchronize` equivalent*

Example :
```java
public class Holder {
	private int value;
	private boolean done;
	private final ReentrantLock lock = new ReentrantLock();
	private final Condition condition = lock.newCondition();

	private void init() {
		lock.lock();
		try {
			value = 12;
			done = true;
			condition.signal(); /* notify() equivalent */
		} finally {
			lock.unlock();
		}
	}

	private void display() throws InterruptedException {
		lock.lock();
		try {
			while (!done) {
				condition.await(); /* wait() equivalent */
			}
			System.out.println(value);
		} finally {
			lock.unlock();
		}
	}
}
```


The `Condition` API allows us to create several conditions for one `ReentrantLock`. This is convenient when we need to wake/sleep a thread for several reasons
	**Associate a reason to a `Condition`**
```java
public class BoundedSafeQueue<V> {
  private final ArrayDeque<V> fifo = new ArrayDeque<>();
  private final int capacity;
  private final ReentrantLock lock = new ReentrantLock();
  private final Condition conditionPut = lock.newCondition();
  private final Condition conditionTake = lock.newCondition();

  public BoundedSafeQueue(int capacity) {
    if (capacity <= 0) {
      throw new IllegalArgumentException();
    }
    this.capacity = capacity;
  }

  public void put(V value) throws InterruptedException {
    Objects.requireNonNull(value);
    lock.lock();
    try {
      while (fifo.size() == capacity) {
        conditionPut.await();
      }
      fifo.add(value);
      conditionTake.signal();
    } finally {
      lock.unlock();
    }
  }

  public V take() throws InterruptedException {
    lock.lock();
    try {
      while (fifo.isEmpty()) {
        conditionTake.await();
      }
      conditionPut.signal();
      return fifo.remove();
    } finally {
      lock.unlock();
    }
  }
}
```

