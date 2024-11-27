[[Java]]
****
**Table of Contents**
```table-of-contents
```

****
## Collection API

The API was introduced with Java 1.2 (in 1998)
	*Before that, we could only use Vector, Stack and Hashtable (most are deprecated now)*

### Interfaces and implementation

The (basic) API looks like this:
![[ESIPE/Semestre 3/Java/06 - Collections/api.png]]

We have two kinds of implementation:
- **Named**: `ArrayList`, `HashSet`, `ArrayDeque`, `HashMap`
- **Anonymous**: `List.of()`, `Arrays.asList()`, `List.subList()`, `Map.copyOf()`

Those interfaces are parameterised (generics).
	`List<E>`, `Set<E>`, `Deque<E>` and `Map<K,V>`
> Each element E/V or key K must implement the following methods:
> - `equals(Object)`
> - `hashCode()`
> - `toString()`

Here is an example of what happens if we don't define `hashCode`:
```java
class Person {
	private int age;
	public Person(int age) { this.age = age; }
	
	public boolean equals(Object o) {
		return o instanceof Person(int age) && age == this.age;
	}
}

var person = new Person(32);
var set = Set.of(person);
System.out.println(set.contains(new Person(32))); // false...
```

As explained [[ESIPE/Semestre 3/Java/01 - Fundamentals/01 - Fundamentals#Encapsulation|here]], we take into account **design by contract**. Once an element is inserted in the collection, we can not modify it anymore:
```java
class Person {
	// ...
	public int hashCode() { return age; }
}

var person = new Person(32);
var set = Set.of(person);
person.age = 23; // mutation, not good
System.out.println(set.contains(person)); // false
```


Should we manipulate a collection by its implementation or its interface ?
```java
// by interface
List<String> list = new ArrayList<>();

// by implementation
ArrayList<String> list = new ArrayList<>()
```

We have a problem for each choice:
- If a method takes an implementation as parameter, we cannot give another
```java
public void m(ArrayList<String> list) { } // Can't give another type of List
```

- An operation on an interface will have a **varying complexity** depending on the underlying implementation
```java
public void m(Set<String> list, String value) {
	set.contains(value);  // HashSet or Set.of(): O(1)
						  // TreeSet: O(ln n)
}
```

The solution is:
- Use the interface for return types and parameters
- Use the implementation for local variables and private fields
```java
public class Library {
	private final ArrayList<Book> books = new ArrayList<>();

	public List<Book> getAllBooksPlus(Book b) {
		var result = new ArrayList<>(books); // var is helpful
		result.add(book);
		return result;
	}
}
```


### Mutation

API uses a same interface for both mutable and immutable collections
	*It avoids having too many interfaces*

This means that methods modifying the collection are optional, as they can raise an `UnsupportedOperationException`.

**We never modify a collection we haven't created**:
```java
static void removeLastIfEmpty(List<String> list) {
	var last = list.get(list.size() - 1);
	if (last.isEmpty()) {
		list.remove(list.size() - 1); // no !
	}
}

static List<String> removeLastIfEmpty(List<String> list) {
	var last = list.get(list.size() - 1);
	if (last.isEmpty()) {
		return list.stream().limit(list.size() - 1).toList(); // ok !
	}
	return list;
}
```

Two types of modifications exists:
- Structural (size change)
	`List.add`, `Set.remove`
- Non-structural
	`List.set`, `Map.replace`
> Some collections only accepts non-structural modifications 


### Constructors and size

A collection must have at least two constructors:
- One with no parameters (create an empty collection)
- One that accepts another collection (copies all its elements into the new collection)
```java
ArrayDeque<E>(Collection<? extends E>)
ArrayList<E>(Collection<? extends E>)
HashMap<K,V>(Map<? extends K, ? extends V>)
```

Enlarging a collection is very costly. For this reason, it is possible to specify a default capacity:
```java
ArrayList<E>(capacity)
<E> HashSet.newHashSet(capacity)
<E> LinkedHashSet.newLinkedHashSet(capacity)
<K,V> HashMap.newHashMap(capacity)
<K,V> LinkedHashMap.newLinkedHashMap(capacity)
```


### View

We can get a view from another collection. A view means that we **keep a pointer on the first collection**, and can access/mutate its content.
```java
var list = new ArrayList<>(List.of(1, 2, 3, 4));
var list2 = list.subList(2, 4); // Elements at index 2 and 3
								// (out boundary is always exclusive in java)
list2.set(0, 42);
System.out.println(list); // [1, 2, 42, 4]. It did change list
```

The concept of view is useful if we want to give **read-only access** to our mutable collection:
```java
class Library {
	private final ArrayList<Book> books;
	public Library(List<? extends Book> books) {
		this.books = new ArrayList<>(books); // defensive copy
	}
	
	public List<Book> books() {
		return Collections.unmodifiableList(books); // read-only view
	}
}
```


### Collections and null

"If we don't store null, we don't receive null"
	*In fact, new collections (Since Java 5) doesn't even allow to store null*

We never return null. If a collection is empty or if what the user searches isn't in our collection, we return an empty one:
```java
public static Set<String> findBlahBlah() {
	if (/* ... */) {
		return null; // bad
	}
	// ...
}

public static Set<String> findBlahBlah() {
	if (/* ... */) {
		return Set.of(); // good !
	}
	// ...
}
```


### Ordering

The way elements are ordered depends on the implementation:
- Insertion order: `SequencedCollection` (`List`, `Deque`, `LinkedHashSet`, `LinkedHashMap`)
- Access order: `LinkedHashMap(capacity, factor, true)`
- Sorted with comparison function: `TreeSet`, `TreeMap`
- No order: `Set`

The `Comparator<T>` functional interface allows to compare two elements of the same type.
```java
interface Comparator<T> {
	int compare(T t1, T t2);
}
```
> Behaviour is similar to `strcmp`:
> \< 0 if t1 < t2
   \> 0 if t1 > t2
   \==0 if `t1.equals(t2)`

**Mind the overflows!**
```java
record Person(int id) {}

// No ! What if id is Integer.MIN_VALUE or close ?
Comparator<Person> comparator = (p1, p2) -> p1.id() – p2.id();

// Good
Comparator<Person> comparator = (p1, p2) -> Integer.compare(p1.id(), p2.id()); 
```


We can define a **natural order** if our class implements `Comparable`:
```java
record Person(int id) implements Comparable<Person> {
	public int compareTo(Person p) { // Must be defined
		return Integer.compare(id, p.id);
	}
}
```


### Concurrency

The `ava.util.concurrent` package provides lock-free implementations:
- List (`CopyOnWriteArrayList`)
- Set (`CopyOnWriteArraySet`, `ConcurrentSkipListSet`)
- Map(`ConcurrentHashMap`, `ConcurrentSkipListMap`)

Those implementations uses [[10 - Atomic Operations#Compare and Set (CAS)|CAS]] or [[10 - Atomic Operations#Volatile|volatile]].
`CopyOnWrite` = Duplicate the array each time we modify it
Skip list = Sorted list (`Comparator`) in O(ln n) complexity


****
## Iterable

Collections implement the `Iterable` interface:
```java
interface Iterable<E> {
	Iterator<E> iterator();
}
```

The `for( : )` syntax allows to use this iterator:
```java
for (var element : elements) {
	// ...
}
```

An Iterator allows to go through a collection in an O(n) complexity. 
We cannot use an index with Iterator, as some collections (`Set` and `Deque`) doesn't have indexes.
	*Furthermore, `List.get(index)` can be O(n)*
```java
LinkedList<Integer> list = // ... Doubly linked
for (var i = 0; i < list.size(); i++) { // O(n2), oops
	var element = list.get(i);
	// ...
}

LinkedList<Integer> list = // ... 
for (var element : list) { // O(n), ok !
	// ...
}
```


`List`, `Queue` and some implementations of `Set` provides a `reversed()` method which returns a reversed [[06 - Collections#View|view]]:
```java
List<String> list = // ...
for (var element : list.reversed()) {
	// ...
}
```
> Since `reversed()` is a view, elements of the collection aren't copied, which offers better performance !


### `Iterator<T>`

The Iterator is the cursor that goes through all of the elements of the collection.
```java
interface Iterator<T> {
	boolean hasNext(); // Checks if there is any element left 
	T next();          // Returns the current element and goes to the next one
}
```

In fact, both are identical:
```java
for (var element : iterable) {
	// ...
}

var iterator = iterable.iterator();
while (iterator.hasNext()) {
	var element = it.next();
	// ...
}
```


### Iterator and mutation

What if we modify the structure of the collection while we are going through it with our iterator ?
- If the collection is immutable: `UnsupportedOperationException` during mutation
- If the collection is concurrent: We don't see the mutation (Snapshot At The Beginning)
- If the collection is mutable: **Next call of `iterator.next()` will crash the program**. This behaviour is called a **Failfast**
	*It will throw a `ConcurrentModificationException` (which is poorly named, as it isn't related to concurrency at all).
	Note that there is no problem in modifying the structure as long as you don't call `iterator.next()` after (so if you just change one value and return, you won't encounter the issue)*

How to solve the problem ?
- We save the mutations to do and apply them when we don't need the iterator anymore
- We create a new collection with a stream
- **We use the methods provided by the Iterator**

An iterator is packed with methods that allows structural changes on the collection he is traversing (no failfast): 
- `Iterator.remove()`: Deletes the element returned by `next()`
- `ListIterator.add()`: Adds an element after the one returned by `next()`
- `ListIterator.set()`: Modifies the element returned by `next()`
> `ListIterator` is an iterator specific to lists


### `forEach()`

The `Iterable` interface also defines a `forEach` method
```java
Iterable<E>.forEach(Consumer<? super E> consumer)
```

So, the collections implements:
- The iterator - **External Iterator**
- The `forEach()` method - **Internal Iterator**

`forEach` is overall more efficient, but appears more limited as lambdas works on effectively final variables.


****
## Contract

Collection interfaces have different contracts.
`Collection<E>` is the base interface, it requires the following methods to be implemented:
- `isEmpty/size()`
- `add(E)`
- `contains(Object)`/`remove(Object)`
- `equals(Object)`/`hashCode()`/`toString()`
- `toArray(intFunction)` and `toArray(T[])`
> **The interface does not specify any order**

The `equals()` method is defined in a strange manner, it is **non-symmetric**.
For the method to return true, both collections must contain the same elements and **the collections must be of the same kind**
	*If we apply `equals()` on a list, the other collection must also be a list*
```java
List.of(1, 2, 3).equals(Set.of(1, 2, 3)) // false
Set.of(1, 2, 3).equals(Set.of(1, 2, 3)) // true
```

`<T> T[] toArray(IntFunction<T[]> function)` takes an array creation function as parameter (e.g., `String[]::new`).
```java
List<Object> list = List.of("foo", "bar");
String[] array = list.toArray(String[]::new);
```


### List

A list **imposes an insertion order**.
It adds the following methods:
- `set(index, E)`, `E get(index)`, `E getFirst()`, `E getLast()`, `add(index, E)`, `addFirst(E)`, `addLast(E)`, `E remove(index)`, `E removeFirst()`, `E removeLast()`
- `int indexOf(E)`, `int lastIndexOf(E)`
- `replaceAll(unaryOperator)`
- `List<E> subList(start, end)`, `List<E> reversed()`
- `sort(comparator)`
> In practice, we don't use `get(index)` and `set(index, E)` as they can have an O(n) complexity. Furthermore, `add(index, E)`, `addFirst(E)`, `remove(index)` and `removeFirst()` are systematically in O(n).

Since Java 21, there are dedicates methods to retrieve first and last elements in O(1):
```java
var first = lire.getFirst();
var first = list.get(0); // before Java 21

var last = list.getLast();
var last = list.get(list.size() - 1); // before Java 21
```
> Each implementation of `List` must have a O(1) guarantee for those two methods


### Set

Collection without duplicates.
It does not provide any additional method than the ones defined in `Collection`.

It allows to find and delete elements in O(1) or O(ln n), which is a good complexity.


### Deque

"**D**ouble **E**nded **Que**ue". It supports element insertion and removal at both ends.
In other words, it can be **used both as a queue or as a stack**.

By default it's a queue, so it adds at the tail and removes at the head (FIFO) 

it provides the following functions:

|             | **First Element (Head)**                                                                        |                                                                                                 | **Last Element (Tail)**                                                                       |                                                                                               |
| ----------- | ----------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
|             | Throws Exception                                                                                | Returns null if impossible                                                                      | Throws exception                                                                              | Returns null if impossible                                                                    |
| **Insert**  | [`addFirst(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#addFirst-E-)     | [`offerFirst(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#offerFirst-E-) | [`addLast(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#addLast-E-)     | [`offerLast(e)`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#offerLast-E-) |
| **Remove**  | [`removeFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#removeFirst--) | [`pollFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#pollFirst--)     | [`removeLast()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#removeLast--) | [`pollLast()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#pollLast--)     |
| **Examine** | [`getFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#getFirst--)       | [`peekFirst()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#peekFirst--)     | [`getLast()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#getLast--)       | [`peekLast()`](https://docs.oracle.com/javase/8/docs/api/java/util/Deque.html#peekLast--)     |
`poll` **extracts** an element from the Deque.
`peek` and `get` **retrieves without removing**.


### Map

A map is a dictionary that associates an unique key to a value.
	*So, no duplicates of key*

It has the following methods:
- `V get(K)`, `V getOrDefault(K, V defaultValue)`, `V put(K, V)`, `V putIfAbsent(K, V)`
- `replace(K, V)`, `replace(K, V, V)`, `remove(Object)`, `remove(Object, Object)`
- `computeIfAbsent(K, Function<K,V>)`: Avoids to re-compute a cached value
```java
class Cache {
	static int myFunction(int value) {
		// ...
	}
	
	private final HashMap<Integer, Integer> map = new HashMap<>();
	
	public int lookup(int value) {
		return map.computeIfAbsent(value, v -> myFunction(v));
	}
}
```

- `merge(K, V, BiFunction<V, V, V>)` can be used like so:
```java
public static Map<String, Integer> group(List<String> list) {
	var map = new HashMap<String, Integer>();
	for(var word : list) {
		map.merge(word, 1, (v1, v2) -> v1 + v2);
	}
	return map;
}
```

- `Set<Map.Entry<K, V>> entrySet()`, `Set<K> keySet()`, `Collection<V> values()`
- `forEach(BiConsumer<K, V>)`


As we can see [[06 - Collections#Interfaces and implementation|on the image here]], Map is not implementing the `Iterable` interface. This is because there is no iterator on a map, we use another mechanism.`

Inside the Map interface lies another interface, called `Map.Entry` (refer to [[03 - Inner Classes & Enumerations#Inner Class|this lesson]]):
```java
interface Map {
	interface Entry<K,V> { // nested
		K getKey();
		V getValue();
		default void setValue(V value) { throw new UOE(); }
	}
	// ...
}
```
> Under the hood, a Map stores information as entries

Since we cannot use an iterator, we acquire the entries via the `entrySet()` method, which returns a **modifiable view** of the key-value pairs:
```java
for (var entry : map.entrySet()) {
	var key = entry.getKey();
	var value = entry.getValue();
	// ...
}
```

With this mechanism, we can also use the `forEach()` method to go through the entries.
	*As always, we are limited to effectively final variables (lambda capture)*

There are also two specific methods available to retrieve only the keys (`keySet()`) or only the values (`values()`).
```java
for (var key : map.keySet()) {
	// ...
}

for (var value : map.values()) {
	// ...
}
```
> If we only need one of the two, it is better to use those methods instead of `entrySet()` which returns both


****
## Implementations

**Lists:**
- `ArrayList<E>`: Dynamic Array
- `Arrays.asList(E..)`: View of an Array
- `List.of(E…)`: Immutable implementation
- `Collections.nCopies(int n, E value)`: n times the same element
- `CopyOnWriteArrayList<E>`: Concurrent implementation

**Maps:**
- `HashMap<K, V>`: Hash Table
- `LinkedHashMap<K, V>`: Same, but keeps the insertion order
- `TreeMap<K, V>`: (Sorted) Red-Black tree
- `Map.of(K, V, ...)`: Immutable implementation
- `EnumMap<K extends Enum<K>, V>`: Keys comes from a (same) enum
- `IdentityHashMap<K, V>`: Uses `==` and `System.identityHashCode()` instead of `equals()` and `hashCode()`
- `ConcurrentHashMap<K, V>`: Concurrent Hash Table
- `ConcurrentSkipListMap<K, V>`: Same, but sorted

**Sets:**
- `HashSet<E>`: Hash Table
- `LinkedHashSet`: Same, but keeps the insertion order
- `TreeSet<E>`: (Sorted) Red-Black tree
- `Set.of(E...)`: Immutable implementation
- `EnumSet<E extends Enum<E>>`: Elements comes from a (same) enum
- `CopyOnWriteArraySet<E>`: Concurrent implementation
- `ConcurrentSkipListSet<E>`: Same, but sorted

**Deques:**
- `ArrayDeque<E>`: Circular Array
- `ConcurrentLinkedQueue`: Concurrent implementation
- `ArrayBlockingQueue`: Blocking concurrent implementation, explained in greater details [[08 - Producer Consumer#BlockingQueue(s)|in this class]]. 


### Write your own implementation

In fact, each of the collection has its respective abstract class: `AbstractList`, `AbstractSet`, `AbstractMap`, `AbstractDeque`
	*It allows an **immutable** implementation*

**List:**
`AbstractList` allows to define an O(1) access. So, `get(index)` is good.
Two methods must be implemented:
- `int size()`
- `E get(index)`
Also, your list must also implement `RandomAccess`, so it indicates that `get()` is in O(1).

We should also define our iterator.
To set up our iterator, we must implement the `Iterable` interface, and define the `iterator()` method that forwards our iterator.
Two methods must be defined: `next()` and `hasNext()`
	*`next()` must raise a `NoSuchElementException` if one attempts to call it when `hasNext()` returns false (try to retrieve the next element when there isn't)*
The best way to do it is through an [[03 - Inner Classes & Enumerations#Anonymous Class|anonymous class]]:
```java
Iterator<String> iterator() {
	return new Iterator<E> () {
		private int i = 0; // Internal counter

		// hasNext should NEVER modify our internal counter
		public boolean hasNext() {
			return i < array.length;
		}

		// next should always begin by checking there is a next element, or throw
		public E next() {
			if (!hasNext()) { throw new NoSuchElementException(); ) 
			var element = array[i];
			i++;
			return element;
		}
	};
}
```

**Set:**
`AbstractSet` is to be used.
Three methods to redefine:
- `int size()`
- `boolean contains(Object)`: This one isn't mandatory, but the default one is in O(n) complexity, so it's better to re-define it...
- `Iterator<E> iterator()`

**Map:**
`AbstractMap` is to be used.
Five methods to implement:
- `int size()`
- `Set<Map.Entry<K, V>> entrySet()`
And three methods to redefine because the complexity is O(n):
- `V get(Object)`
- `V getOrDefault(Object, V defaultValue)`
- `boolean containsKey(Object)`


****
## Legacy

Here is a more complete overview of the API:
![[api-complete.png]]

Some of those interfaces (and some implementations) are:
- Not used much
- Have been replaced (bad performance or algorithm)

**Most important ones:**
`LinkedList`: Doubly linked list, too slow compared to implementations using an array.
	*Replace it with an `ArrayList` or an `ArrayDeque` (with insertion at the beginning)*

`PriorityQueue`: Allows to sort elements with a heap, but the algorithm is unstable
	*Two `equal()` objects can be inserted in one order and retrieved in another order...*

Pre-1.2 classes (before Collections):
- `Vector` -> `ArrayList`
- `Stack` -> `ArrayDeque`
- `Hashtable` -> `HashMap`
- `Enumeration` (not related to Enums) -> `Iterator`
Furthermore, all those classes used a `synchronize` for their methods
	*Slow if we don't need concurrency, slow compared to the lock-free implementations provided in the dedicated collections in `java.util.concurrent`, and dangerous when using an iterator*

