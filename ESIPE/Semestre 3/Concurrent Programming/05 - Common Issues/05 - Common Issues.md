[[Concurrence]]
****
**Table of Contents**
```table-of-contents
```

****
## Deadlocks

What does this code prints ?
```java
public class PingPong {
	private final Object lock1 = new Object();
	private final Object lock2 = new Object();

	public void ping() {
		synchronized (lock1) {
			synchronized (lock2) { System.out.println("ping"); }
		}
	}

	public void pong() {
		synchronized (lock2) {
			synchronized (lock1) { System.out.println("pong"); }
		}
	}

	public static void main(String[] args) throws InterruptedException {
		var pingpong = new PingPong();
		Thread.ofPlatform().start(() -> {
			for (;;) { pingpong.ping(); }
		});
		for (;;) { pingpong.pong(); }
	}
}
```
> [!bug]
> Sometimes ping and pong, sometimes ping only, sometimes pongs only, sometimes nothing
**But most of the time, it will be blocked !**

What happens:
- ping thread takes the token for `lock1`
- ping thread is de-scheduled right after, before taking the `lock2` token

- pong thread takes the token for `lock2`
- pong thread is de-scheduled right after, before taking the `lock1` token

- ping thread is re-scheduled and attempts to grab `lock2`'s token, which he cannot because the other thread already holds it
- ... and pong thread will only release it if he can grab the `lock1`'s token, which the other thread holds

We are in a deadlock situation.
	*This is very common when threads takes several locks in an interlaced order*

> [!important] How to avoid this situation ?
>1. Only use one lock
>2. If not, always take the locks in the same order
>3. Use a **lock-free** programming philosophy, which uses `volatile` and `Atomic` operations. This is way too complex for now, and will be seen in S4.
>
> If the threads always take the locks in the same order, a **deadlock situation can never happen**.


****
## Constructor and publication issues

```java
public class A {
	private final int value; /* final! */
	private static A a;

	public A(int value) {
		this.value = value;
	}

	public static void main(String[] args) {
		Thread.ofPlatform().start(() -> {
			// displays 7 or nothing
			if (a != null) {
				System.out.println(a.value); 
			}
		});
		a = new A(7);
	}
}
```

`final` means an object's property is visible with its initialisation value by all other threads **at the end of the constructor**. Also, this value can not be changed afterwards, it is definitive.

If we forgot the `final` keyword, it is possible for the second thread to print 7, nothing, **or 0 (default value of an integer)**

> [!important]
> For this reason, it is important to set a property to `final` when possible. 
> If non-final properties must be declared in the constructor, **always use a `synchronize` block in it**.

> [!info]
> Setting value of a final property directly in it's declaration works the same as defining it in the constructor.
> ```java
> // Those two are doing the same
> public class A {
> 	private final int value;
> 
> 	public A() {
> 		this.value = 5;
> 	}
> }
> 
> public class A {
> 	private final int value = 5;
> 
> 	public A() { }
> }
> ```
> 

There is no publication issues with `static` properties initialisations.


**Summary**

Never launch a thread in a constructor

No need for `synchronize` in the constructor if:
- Fields are final
- Field is initialised with its default value (e.g. an int with a value of 0)


