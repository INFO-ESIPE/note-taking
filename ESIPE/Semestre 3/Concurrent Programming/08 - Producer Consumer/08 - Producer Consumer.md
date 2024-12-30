[[Concurrence]]
****
**Table of Contents**
```table-of-contents
```

****
## Producer Consumer

Producer/Consumer is an intuitive **Design Pattern** that **allows one or several threads producing data to communicate with one or several threads processing data**.

This pattern makes it easy to solve two major issues:
- Stop consumers when there is no data to process (no busy waiting)
- Stop producers when all consumers are currently too busy processing data


![[design.png]]

We use a **blocking queue**.
	*When the queue is full, producers waits for it to empty.*


****
## BlockingQueue(s)

`BlockingQueue` is a native interface provided in the concurrent package. This interface has several implementations:
- `ArrayBlockingQueue`: Acts as an `ArrayDeque` (circular array)
- `LinkedBlockingQueue`: Acts as a Linked List (size is not set by default)
	*This one is to avoid, as it can cause an `OutOfMemoryError` if not handled correctly (which is like the worst error than can occur)*
- `SynchronousQueue`: Only accepts one item, and DOES NOT STORE IT


As `BlockingQueue` is an interface, those three implementations shares the same methods:

|             | _Throws exception_ | _Special value_ | _Blocks_     | _Times out_            |
| ----------- | ------------------ | --------------- | ------------ | ---------------------- |
| **Insert**  | `add(e)`           | `offer(e)`      | **`put(e)`** | `offer(e, time, unit)` |
| **Remove**  | `remove()`         | `poll()`        | **`take()`** | `poll(time, unit)`     |
| **Examine** | `element()`        | `peek()`        | _-_          | _-_                    |
> [!important]
> In a Producer/Consumer pattern, we **NEVER use the methods raising exceptions**, and **ALMOST NEVER uses the ones returning special values***

```java
var queue = new ArrayBlockingQueue<String>(10);

for (var i = 0; i < 3; i++) {
    Thread.ofPlatform().start(() -> {
        for(;;) {
            try {
                Thread.sleep(100);
                queue.put(Thread.currentThread().getName());
            } catch (InterruptedException e) {
                return;
            }
        }
    });
}

for (var i = 0; i < 2; i++) {
    Thread.ofPlatform().start(() -> {
        for(;;) {
            try {
                Thread.sleep(500);
                System.out.println("next : " +  queue.take());
            } catch (InterruptedException e) {
                return;
            }
        }
    });
}
```

