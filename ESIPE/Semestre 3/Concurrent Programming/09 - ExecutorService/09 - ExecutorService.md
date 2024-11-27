[[Concurrence]]
****
**Table of Contents**
```table-of-contents
```

****
## `ExecutorService`

We still have two problems with concurrency so far:
- Platform threads (OS resources) are costly and slow to start
- It is difficult to collect a value calculated by a thread

A solution would be to **re-use our threads for several tasks**.
Tasks will become `Callable<T>` instead of `Runnable` (as it now returns a value):
```java
@FunctionalInterface
public interface Callable<T> {
    T call() throws Exception;
}
```


When a task is submitted, it returns a `Future<T>`, which allows us to retrieve the calculated value at a later time.

We obtain something that will look like this:
![[executorservice.png]]


****
## `ExecutorService` creation

We build an `ExecutorService` through the provided factory methods provided:
- `Executors.newFixedThreadPool(int poolSize)` - Starts `poolSize` worker-threads which are **never stopped.**
- `Executors.newCachedThreadPool()` - Starts as much worker-threads as necessary, but attempts to re-use them. **If a worker-thread is inactive since a minute, it is stopped.** 
- `Executors.newSingleThreadPool()` - Starts a single thread that is never stopped
	*Equivalent to `Executors.newFixedThreadPool(1)`*

In any case, the `BlockingQueue` used to store the tasks (when threads are busy working) **has no limits**.


It is possible to go for a more granular and manual approach with a `ThreadPoolExecutor`. Its constructor takes five parameters:
- `int corePoolSize`: Number of thread we expect to use at a normal regime.
- `int maximumPoolSize`: Maximum allowed amount of alive threads.
- `long keepAliveTime` : If there are more than `corePoolSize` alive threads, those who exceeds are terminated after `keepAliveTime * unit` of inactivity.
- `TimeUnit unit` : Inactivity time unit.
- `BlockingQueue<Runnable> workQueue` : BlockingQueue implementation to use in order to store tasks if all worker-threads are busy.
	*In fact, the factory methods are just pre-configured versions of this.*


An ExecutorService can be closed in two ways:
- `shutdown()`: Prevents submission of new tasks and stop threads when all remaining tasks are completed
- `shutdownNow()`: Attempts to interrupt all worker-threads. However, if some doesn't answer to the `interrupt()` signal, they will [[07 - Interruptions and Exceptions#Stop a Thread|never be stopped]]. 


****
## `ExecutorService` with `Future<T>`

We can submit a task via the `submit()` method (two ways):
```java
<T> Future<T> submit(Callable<T> callable); // callable

Future<?> submit(Runnable run) // Since a runnable returns nothing, the Future
							   // has no type
```

Those methods are **non-blocking**, this is the whole purpose of the `Future<T>` object that is returned immediately (and from which we can collect the true value once the `Callable<T>` has returned it).


`java.util.concurrent.Future` represents the result of an asynchronous computation, allowing us to retrieve the value once it has arrived at a later time (in the future...)


**Retrieve the value**
In two ways:
- `future.get()`: **Blocking method** until the `Callable<T>` has returned the value.
	*If any Exception is thrown during the `Callable<T>`'s task, calling this method will throw a `java.util.concurrent.ExecutionException`*

- `future.resultNow()`: **Not blocking**, Immediately returns the result of the calculation.
	*If one calls this method when the `Callable<T>` has not yet returned anything, an `ExecutionException` is returned... Be careful*
```java
public static int bigComputation(int j) { // Syracuse
	var i = 0;
	while (j > 1) {
		j = j % 2 == 0 ? j = j / 2 : 3 * j + 1;
		i++;
	}
	return i;
}

public static void main(String[] args) throws InterruptedException {
	var executorService = Executors.newFixedThreadPool(2);
	var future = executorService.submit(() -> bigComputation(3333));
	try {
		System.out.println(future.get());
	} catch (ExecutionException e) {
		throw new AssertionError(e.getCause());
	}
	// ...
}
```


**Cancel a task**
The `future.cancel(boolean interrupt)` method allows to ask for the task's interruption.
- `cancel(true)`: Attempts to **interrupt the task if its currently running**.
- `cancel(false)`: Allows the task to complete if it has already started, but **prevents new tasks from beginning**.


**Check task status**
The `future.state()` method allows to check the current state of the task:
- `RUNNING`: Currently being computed **or awaiting in the queue**
- `SUCCESS`: Task completed without throwing any exception.
- `FAILED`: Task did throw an exception.
- `CANCELLED`: Task cancelled via the `future.cancel()` method.


****
## Invoke

In fact, we often just use one of those two methods to manipulate our tasks:
- `invokeAll`, which takes a collection of `Callable<T>`, blocks until they **ALL OF THEM** has been executed (whatever the result is), and returns a list of `Future<T>`.
- `invokeAny`, which takes a collection of `Callable<T>`, blocks until **ONE OF THEM** has been executed (without throwing an exception) and returns the value forwarded by the `Callable<T>`
	*Here, we do not need a `Future<T>` to manage a possible Exception, since we know that the `Callable` completed successfully...*


**Example for `invokeAll` (pre-Java19):**

```java
var executorService = Executors.newFixedThreadPool(2);

var callables = new ArrayList<Callable<Integer>>();
IntStream.range(1, 100).forEach(i -> callables.add(() -> bigComputation(i)));

var futures = executorService.invokeAll(callables);
try {
	for (var future : futures){
		System.out.println(future.get());
	}
} catch (ExecutionException e) {
	throw new AssertionError(e.getCause());
}
```


**Example for `invokeAll` (post-Java19):**

This new approach uses pattern-matching (better):
```java
var executorService = Executors.newFixedThreadPool(2);

var callables = new ArrayList<Callable<Integer>>();
IntStream.range(1, 100).forEach(i -> callables.add(() -> bigComputation(i)));

var futures = executorService.invokeAll(callables);
for (var future : futures) {
	switch (future.state()) {
      case RUNNING -> throw new Assertion("should not be there");
      case SUCCESS -> System.out.println(future.resultNow());
      case FAILED -> System.out.println(future.exceptionNow());
      case CANCELLED -> System.out.println("cancelled");
	}
}
```


**Example for `invokeAny`:**

```java
var executorService = Executors.newFixedThreadPool(2);

var callables = new ArrayList<Callable<Integer>>();
IntStream.range(1,100).forEach(i -> callables.add(() -> bigComputation(i)));

try {
    System.out.println(executorService.invokeAny(callables));
} catch (ExecutionException e) {
    throw new AssertionError(e);
}
```
