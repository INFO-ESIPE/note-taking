[[Concurrence]]
****
**Table of Contents**
```table-of-contents
```

****
## Paradigms

- Concurrency: Running multiple calculations simultaneously (what we are doing since the beginning of this class).
- **Parallelism**: Splitting a calculation across multiple threads.
- Vectorization (Vectorized Operations): Executing a single operation (e.g., addition) on multiple data points in memory using a single thread.
- Distributed Processing: Spreading a calculation across multiple machines.


****
## Start / Join

For parallel computing, we used to operate with a **Start / Join** model (based on the `fork()` / `waitpid()` model for Unix System programming):
```java
var threads = IntStream.range(0, 5)
	.mapToObj(i -> new Thread(() -> {
		var startIndex = ...
		var endIndex = ...
		for(var i = startIndex; i < endIndex; i++) {
			array[i] *= 2;
		}
	})).toList();
for(var thread: threads) { thread.start(); }
for(var thread: threads) { thread.join(); }
```

It will look like this (on an array of 48 cells):
![[startjoin.png]]


This model works fine, as long as:
- We know all the **data upfront**
	*Works well on an array, but less for a Red-black tree (`TreeMap` in Java)*
- If the calculation duration is homogeneous
	*If one thread takes way longer to perform an action, it will block the calculation while other threads have already finished their task a while ago
	**Calculation distribution issue!***


****
## Fork / Join

A naive approach that could solve the first issue would be a simple "divide and conquer", which doesn't require us to know the data upfront anymore:
```java
// pseudo-code...
process(int startIndex, int endIndex) {
	// SMALL = size of smallest task
	if (endIndex – startIndex < SMALL) {
		// for(...)
		return;
	}
	
	var middle = (startIndex + endIndex) >>> 1;
	start process(startIndex, middle);
	start process(middle, endIndex);
	join
	join
}
```

We can make this model even better by avoiding pending threads: **Fork / Join**
```java
// pseudo-code...
process(int startIndex, int endIndex) {
	if (endIndex – startIndex < SMALL) {
		// for(...)
		return;
	}
	
	var middle = (startIndex + endIndex) >>> 1;
	fork process(startIndex, middle);
	var result2 = process(middle, endIndex);
	var result1 = join
	return combine result1 result2
}
```

It will look like this (on an array of 48 cells):
![[forkjoin.png]]


****
## Fork / Join with Work Stealing

This is good, but our second issue remains.
The only solution is to decouple the calculation part from the thread performing the calculation. We will work with a **pool of thread**, and introduce the concept of **work stealing**:
- The number of thread (pool size) is fixed (similar to a `ThreadPoolExecutor`)
- Each thread has a queue of task (similar to a `BlockingQueue`)
- If a thread **finished computing all tasks in his queue**, he will **steal tasks at the end of other thread's queue**

This capacity of stealing tasks to others helps balancing the workload.

It will look like this (on an array of 48 cells):
![[workstealing.png]]


****
## Configure Fork / Join

As we have seen [[13 - Fork Join#Fork / Join|above]], we need to set a static `SMALL` threshold that defines **the size of the smallest task** that should be processed. We need to set this value correctly:
- Too big: Not enough tasks, so not enough parallelism
- Too small: Too much tasks, which results in a lot of allocations and an accumulation of tasks in the queues (dangerous)

Also, the `combine` instruction must be an **associative operation** to ensure correct results across splits.
	*We cannot split the calculation if `combine(combine(a, b), c) != combine (a, combine(b, c))`...*

We need to look into it...


****
## Fork / Join in Java

We will use the `java.util.concurrent.ForkJoinPool` class to represent our pool.
	*As I've mentioned, this class is literally extending the `AbstractExecutorService` class, and implements a **work-stealing** feature on it...
```java
/* Create our own custom and separate pool (pool size of 5 threads) */
ForkJoinPool pool = new ForkJoinPool(5);

/* Get the instance of the "common pool". If a ForkJoinTask isn't submitted to any specific pool, it will be executed here. It is usually more performant than a manual pool as it reduces resource usage (threads are slowly reclaimed during periods of non-use, and reinstated upon subsequent use) */
ForkJoinPool commonPool = ForkJoinPool.commonPool();
```

It executes (and manages) `ForkJoinTask` which can either be:
- `RecursiveAction`: returns no value
- `RecursiveTask<T>`: returns a result with the type indicated in the diamond syntax

We can execute a task on the pool via:
- `execute(RecursiveAction)`: Asynchronous task
- `invoke(RecursiveAction|RecursiveTask)`: Synchronous (blocks the current thread until completion)
- `submit(RecursiveAction|RecursiveTask)`: Returns a Future, explained [[09 - ExecutorService#`ExecutorService` with `Future<T>`|here]]

*When doing a parallel stream (`stream.parallel()`), it actually use the `ForkJoinPool.commonPool()` under the hood (default pool), employing recursive decomposition via `Spliterator.trySplit()` to distribute tasks in parallel.*


****
## Examples

**Fibonacci**

Let's take this stupid (but simple) recursive calculation:
```java
int fibonacci(int n) {
	if (n <= 1) {
		return n;
	}
	return fibonacci(n – 1) + fibonacci(n – 2);
}
```

Now, let's parallelise the calculation with a Fork / Join pattern:
```java
private static class Fibonacci extends RecursiveTask<Integer> {
	private final int n;
	private Fibonacci(int n) { this.n = n; }
	
	@Override
	protected Integer compute() {
		if (n <= 1) {
			return 1;
		}
		
		var fib1 = new Fibonacci(n - 1);
		var fib2 = new Fibonacci(n - 2);
		f1.fork(); // sends to the ForkJoinPool
		var result2 = fib2.compute(); // calculates in current thread
		var result1 = f1.join(); // awaits result
		return result1 + result2; // + is associative
	}
}
// ... 
var pool = ForkJoinPool.commonPool();
var fibo = new Fibonacci(15);
var result = pool.invoke(fibo); // awaiting end of calculation
```


**Multiply values of an array by 2**

We get the following code:
```java
private static class MultiplyBy2 extends RecursiveAction {
	private final int[] array;
	private final int start;
	private final int end;
	private MultiplyBy2(...) { this.array = array; this.start = start; this.end = end; }
	
	@Override
	protected void compute() {
		if (end – start < 1024) {
			for(var i = start; I < end; i++) { array[i] *= 2; }
			return;
		}
		
		var middle = (start + end) >>> 1;
		var mul1 = new MultiplyBy2(array, start, middle);
		var mul2 = new MultiplyBy2(array, middle, end);
		mul2.fork();
		mul1.compute();
		mul2.join();
	}
}
// ...
var pool = ForkJoinPool.commonPool();
var fibo = new MultiplyBy2(array, 0, array.length);
var future = pool.submit(fibo); // get a future
// ...
System.out.println(future.get()); // retrieve the value
```