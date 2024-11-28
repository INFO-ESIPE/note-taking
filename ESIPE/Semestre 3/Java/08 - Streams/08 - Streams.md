[[Java]]
****
**Forewords:** Basic information about streams are already detailed [[02 - Lambdas#Stream|here]]. We will take them for granted.
****
**Table of Contents**
```table-of-contents
```

****
## Parallel Stream

It is possible to parallelise a stream. The calculation is done—pretty much—in the same way whether it is sequential or parallel.
```java
collection.parallelStream(); // First way of retrieving
Stream.parallel() // Second way
```
> Note that a parallel stream is not necessarily quicker than a sequential stream (Distributing the calculation, aggregating results... takes time). This is especially true if the calculation depends on an order and if we have to store intermediate state of the calculation.

Here are some operations which aren't efficient on parallel streams:
- `distinct()`/`sorted()`: This makes the overall thing stateful.
	*Also, `distinct` is always a gluttonous operation, even in SQL*
- `limit()`: Used along with `unordered()` fixes the performance issue, as it indicates to the stream that we do not mind about the ordering
- `findFirst()`: We use `findAny()` instead


Using `forEach` on a parallel stream is doing weird things. A solution to this is to use [[02 - Lambdas#Collectors|Collectors]]:
```java
List<String> list = // ...
ArrayList<String> result = new ArrayList<>();
list.parallelStream()
	.map(String::toLowerCase)
	.forEach(result::add); // weird behaviour

// Solution
list.parallelStream()
	.map(String::toLowerCase)
	.collect(Collectors.toList()); // better
```


****
## Spliterator

What if we have defined our own collection (cf. [[06 - Collections|previous class]]), and we want to use streams on it.
We could implement the Stream interface, but this forces us to define like 30+ methods which is kind of a lot yeah.
Instead, we will define a **Spliterator**, which is the abstraction behind a Stream.
	*We can retrieve a stream from a spliterator, and the later only requires us to implement 4 methods, which is a more reasonable amount...*
```java
Spliterator<String> spliterator = // ...
boolean parallel = false;
Stream<String> stream = StreamSupport.stream(spliterator, parallel); // here
```

As we know, a stream is an abstraction of a "stream" (sequence) of element. This part is actually done by the spliterator. A Spliterator is actually:
- A **push Iterator** (unlike `Iterator` which is a **pull iterator**)
- which can split itself in two
- and remembers characteristics of the stream source in order to optimise the calculation
	*For instance, we can specify that our data is ordered, contains no null values, we can also tell him the exact size of the data so it doesn't have to calculate it... All this knowledge allows the stream to be more efficient.*

Here are the four methods we need to define if we implement the `Spliterator<T>` interface:
- `int characteristics()`: Returns the characteristics of the spliterator (optimisations)
- `long estimateSize()`: As the data has to be separated in two equal parts, we need to know the size. If we can't efficiently compute the size (or if it is very large, close to infinite), we chose `Integer.MAX_VALUE`
- `Spliterator<T> trySplit()`: Attempts to split into two parts. It either returns the over half, or null
- `boolean tryAdvance(Consumer<? super T> action)`: If at least one element remains:
1. Calls the the consumer on the current element
2. Goes to the next one (equivalent to calling `next()` on a pull iterator)
3. If it could find a next element, returns true. Otherwise, false (no element left)

The characteristics (defining the set of data we manipulate) are the following:
- `IMMUTABLE`: The source is immutable
- `NONNULL`: The source does not contain any null element
- `DISTINCT`: Elements are unique
- `ORDERED`: Elements are ordered
- `SORTED`: Ordering follows a sort order
- `SIZED`: We know the size of the data we manipulate
	*In other words, value returned by `estimateSize` prior to any split is a finite and exact count of the elements in the collection (that we would encounter in a complete traversal)*
- `SUBSIZED`: We know the size (`SIZED`) after one or several call to `trySplit()`
- `CONCURRENT`: The source may be *safely* concurrently modified

Two additional methods exists, but aren't mandatory:
- `forEachRemaining(Consumer<? super T> consumer)`: Does a traversal of remaining element, like a `forEach`...
- `Comparator<? super T> getComparator()`: If the collection is sorted by a comparator (e.g. `TreeSet`), it returns this comparator.
	*If the source is sorted in a natural order, null is returned. If not sorted, it throws an `IllegalStateException`. The default method always throws the exception, so you need to redefine it if you want to correctly provide the comparator.*


### Example: From an Array

```java
@SafeVarargs
public static <T> Spliterator<T> fromArray(int start, int end, T... array) {
	return new Spliterator<T>() {
		private int i = start;
		
		@Override
		public Spliterator<T> trySplit() { 
			var middle = (i + end) >>> 1;
			if (middle == i) {
				return null;
			}
			var spliterator = fromArray(i, middle, array);
			i = middle;
			return spliterator;
		}
		
		@Override
		public boolean tryAdvance(Consumer<? super T> consumer) {
			if (i < end) { 
				consumer.accept(array[i++]); 
				return true; 
			}
			return false;
		}
		
		@Override
		public int characteristics() { return SIZED; }
		
		@Override
		public long estimateSize() { return end - i; }
	};
}
```


****
## Optional

Coming from the functional programming world, we use `Optional` when the result of a calculation **might not exist**.
	*A problem we often see in poorly designed code: functions tends to return random null values in some scenarios, which causes a lot of issues later on in the code (late `NullPointerException` is the main one, a typical issue among beginners). 
	Try to imagine yourself using this code as a library, and noticing that the code returns null values when its not able to compute things.
	Instead of forcing the user to read each function's javadoc to ensure it will not return null values, we simply indicate that the method returns an `Optional<T>`, so he clearly knows that the expected value might not be present.*

**Example:** A `findFirst(List<E>)` method which returns the first element of a list should, in fact, return an `Optional<E>`, as the list might be empty.

We can create an `Optional<E>` like so:
- `Optional.empty()`: An optional without any value
- `Optional.of(E Element)`: An optional containing a non-null value
- `Optional.ofNullable(E elementOrNull)`: An optional containing a value, which can be null or not
> There are also Optional for primitive types (`Optionalint` ...)


### Retrieve

So, `Optional` is like a box that might or might not contain something inside. We need a way to open this box without crashing the application.

**We should always check the value's existence before retrieving it:**
We use `isEmpty` or `isPresent`.

**Basic retrieving methods:**
There are several ways to retrieve the value safely or handle the absence of a value.
- `get()` / `orElseThrow()`
    - Use `get()` or `orElseThrow(Supplier<? extends Throwable>)` ==when you’re sure the value is present.==
    - If the value is missing, these throw a `NoSuchElementException` (for `get()`) or a custom exception (for `orElseThrow(Supplier)`).
    - **Be cautious when doing this, other retrieving methods are preferable** 
```java
// This is unsafe, if the Optional doesn't contain something, it crashes.
// So, we need a clean way to handle this on the upper method
Optional<String> opt = // ... 
String value = opt.orElseThrow(() -> new IllegalStateException("No value found"));
```

- `orElse(T other)`
	- Returns the value if present; otherwise, returns the default value provided as `other`.
```java
Optional<String> opt = Optional.empty(); 
String value = opt.orElse("default"); 
System.out.println(value); // "default"
```

- `orElseGet(Supplier<? extends T>)`
	- Similar to `orElse`, but instead of a fixed default value, it uses a supplier to compute the default value **lazily** (only if needed).
```java
Optional<String> opt = Optional.empty(); 
String value = opt.orElseGet(() -> computeDefault()); 
System.out.println(value); // Outputs the computed default value
```


We can also do direct operations on the Optional, which is cleaner:
```java
// Boilerplate
Optional<String> opt = // ...
if (opt.isPresent()) {
	System.out.println(opt.orElseThrow());
}
// Better: void ifPresent(Consumer<? super T>)
Optional<String> opt = // ...
opt.ifPresent(System.out::println);

// Boilerplate
OptionalInt result = // ...
if (result.isPresent()) {
	System.out.println("result is " + result.orElseThrow());
} else {
	System.out.println("result not found");
}
// Better: Optional<R> map(Function<? super T, ? extends R>) + orElse
OptionalInt result = // ...
System.out.println(result
	.map(value - > "result is " + value)
	.orElse("result not found"));
```


### (Not) Storing Optionals

We don't store Optional values, as this both alters performance and cause poor code design.
	*Optional is used for calculation raw output. The user should deal with this temporary state and retrieve a normal value before storing it.*

But let's say we have a class that is built on an Optional. Here is what we can do:
```java
// Bad: we store Optional
public class Foo {
	private final Optional<Bar> bar;
	public Foo(Optional<Bar> bar) { this.bar = bar; }
	public Optional<Bar> getBar() { return bar; }
}

// Better: Via encapsulation, we store an internal null that we pass as an optional
public class Foo {
	private final Bar bar; // might be null
	
	public Foo(Optional<Bar> bar) {
		this.bar = bar.orElse(null);
	}

	public Optional<Bar> getBar() {
		return Optional.ofNullable(bar);
	}
}
```

As we don't like to store things that might not exist, we also don't make collections of Optional:
```java
// Not good, we are making a collection of things that possibly does not exist
// That's pretty silly when you think about it for a second
List<Foo> foos = // ...
List<Optional<Bar>> bars = foos.stream()
	.map(Foo::getBar)
	.collect(Collectors.toList())

// Good: We use flatMap to get rid of the optionals
// (by turning them into a stream)
List<Foo> foos = // ...
List<Bar> bars = foos.stream()
	.flatMap(foo - > foo.getBar().stream()) // here
	.collect(Collectors.toList())
```